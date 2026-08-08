# DDP v0.2 — Pipeline de importación de telemetría iRacing

> Documento de diseño de la etapa v0.2.
> Se deriva de `ARCHITECTURE.md`, `DATA_MODEL.md`, `DATABASE_DESIGN.md` y
> `DDP_RULES.md`, todos **congelados**. Este documento no los modifica.
>
> Ante cualquier conflicto con un documento congelado, el documento congelado
> tiene prioridad y este diseño se corrige.

---

## Estado

| Aspecto | Situación |
|---------|-----------|
| Versión | v0.2 |
| Estado | **Congelado — etapa implementada y cerrada** |
| Esquema SQLite | Sin cambios. Se usa la capa de persistencia de v0.1 tal cual |
| Congelado | Sí. 08/08/2026 |
| Implementación | Completa (fases 0–7). Ver `CHANGELOG.md` y `verify_v0_2.py` |

**Versión 1.0 · congelada · referencia oficial para la implementación del
pipeline de importación iRacing (v0.2).**

Ningún cambio estructural durante la implementación. Si aparece una necesidad
arquitectónica real, se detiene el desarrollo y se abre una revisión de este
documento — la misma regla que cerró `DATABASE_DESIGN.md` en v0.1.

---

# 1. Objetivo

Diseñar e implementar el **pipeline completo** que convierte un fichero de
telemetría de iRacing (`.ibt`) en hechos permanentes de DDP:

```
bytes del .ibt
    → TelemetryImport (Core)
    → resolución de catálogo y piloto (Backend)
    → lote atómico de importación (Backend + SQLite)
    → Session · Lap · TelemetryFile persistidos
```

Al cerrar v0.2, DDP debe poder:

1. Recibir uno o varios `.ibt` desde el escritorio.
2. Reconocer si el fichero ya se importó (`content_hash`).
3. Extraer metadatos de sesión, vueltas y canales.
4. Resolver (o crear por resolución) circuito, coche, piloto y contexto de
   catálogo.
5. Persistir la sesión, sus vueltas y el registro del fichero en una sola
   operación atómica.
6. Dejar el fichero disponible en disco bajo una ruta gestionada.
7. Auditar qué entró, cuándo y con qué resultado (`import_batch`).
8. Reimportar el mismo fichero sin duplicar hechos.
9. Importar telemetría de otro piloto iRacing como `Driver.kind = external`,
   usable después como referencia.

Lo que **no** hace v0.2: analizar rendimiento, generar informes, proponer
objetivos ni interpretar conducción. Eso es v0.3 (Core de análisis) en adelante.

---

# 2. Alcance

## 2.1 Dentro de v0.2

| Pieza | Descripción |
|-------|-------------|
| Lectura `.ibt` | Header binario, session info YAML, canales escalares, marcas de vuelta |
| Contrato `TelemetryImport` | Única salida pública del Core hacia el Backend |
| Orquestación de importación | Backend como adaptador: archivos, catálogo, persistencia |
| Almacenamiento de ficheros | Copia gestionada + huella + disponibilidad |
| API de importación | Endpoints mínimos para disparar y consultar importaciones |
| UI mínima | Selección de fichero(s), progreso, resultado y errores |
| Verificación | Suite por fase + regresión de extremo a extremo |

## 2.2 Fuera de v0.2

| Pieza | Motivo | Etapa prevista |
|-------|--------|----------------|
| Análisis (`telemetry/analyzer.py` y hermanos) | Mide e interpreta; necesita hechos ya importados | v0.3 |
| Garage61 (CSV por vuelta, API) | Segunda fuente; el modelo ya la admite (M5), pero no es iRacing nativo | Posterior a v0.2 |
| Setup `.sto` | Entidad futura `Setup` (9.1 del modelo) | Cuando se abra esa entidad |
| Sectores oficiales | Entidad futura `Sector` (9.2 del modelo) | Idem |
| Resultados sin telemetría | Importar solo clasificación/resultado sin `.ibt` | Roadmap / fase posterior |
| Prioridad entre fuentes | Concepto aceptado en `ROADMAP.md` §2; con una sola fuente no aplica | Cuando haya varias fuentes |
| Versionado de catálogo | Concepto aceptado en `ROADMAP.md` §1 | Después de importar datos reales |
| Ingeniero IA / informes / scoring | Fuera del pipeline de importación | v0.3–v0.4 |

## 2.3 Frontera con la infraestructura (v0.1)

La infraestructura queda **cerrada**:

- No se modifican `ARCHITECTURE.md`, `DATA_MODEL.md`, `DATABASE_DESIGN.md`,
  `DDP_RULES.md`.
- No se modifican migraciones ni el esquema SQLite.
- No se modifica la capa `backend/db/*` salvo **correcciones de errores
  demostrables** (fallo reproducible + prueba que lo pincha).
- El pipeline de v0.2 **consume** `ImportLog`, `SessionRepository`,
  `CatalogRepository`, `DriverRepository` y `Resolution` tal como existen.

Si al implementar aparece un atributo de dominio que no cabe en el modelo, se
detiene el trabajo y se abre una versión 1.1 del documento correspondiente. No
se improvisa una columna.

---

# 3. Principios que gobiernan el diseño

Heredados de los documentos congelados; aquí solo se aplican al pipeline.

| # | Principio | Consecuencia en v0.2 |
|:-:|-----------|----------------------|
| P1 | Frontera Core / Backend estricta | Apartado 4. El Core transforma bytes → `TelemetryImport` completo, o falla con error estructurado (D12) |
| P2 | Solo el Core interpreta el binario `.ibt` | El Backend no parsea telemetría; obtiene bytes y llama al Core |
| P3 | El fichero nunca entra en SQLite | Solo metadatos + `storage_path` + `content_hash` |
| P4 | Toda importación es atómica | Un fallo deja la base exactamente como estaba |
| P5 | Ninguna importación pisa en silencio | Se rellena lo vacío; corregir exige petición explícita |
| P6 | Un dato, un lugar | iRating solo en `session_metric`; "Pole" no se almacena |
| P7 | Hechos inmutables tras importar | Reimportar no reescribe una sesión ya consolidada |
| P8 | Deduplicación por huella | `telemetry_file(content_hash)` es el camino caliente |
| P9 | Procedencia por lote | Una fila `import_batch` por operación, con registro original |
| P10 | El parser traduce, no opina | Ni deltas, ni insights, ni scoring en v0.2 |

---

# 4. Arquitectura del pipeline

## 4.1 Frontera Core / Backend (explícita e innegociable)

Esta separación es la regla de oro de v0.2. Si una responsabilidad cae del
lado equivocado, el diseño está mal aplicado.

### El Core

El Core **únicamente** transforma:

```
bytes del archivo .ibt
        ↓
TelemetryImport
```

El Core **no conoce**:

| Prohibido en el Core | Motivo |
|----------------------|--------|
| SQLite / conexiones / SQL | La persistencia es del Backend |
| FastAPI / HTTP / status codes | El transporte es del Backend |
| Repositorios (`ImportLog`, `SessionRepository`, …) | Son adaptadores de infraestructura |
| Rutas de disco / pathlib de almacén / `app_setting` | El Backend gestiona el sistema de archivos |
| GitHub / red / APIs externas | El Core no tiene I/O de entorno |
| La aplicación (Tauri, UI, ciclo de vida) | El Core es un motor reutilizable |
| Piloto activo, preferencias, tema | Contexto de instalación, no de dominio de parseo |

El Core **sí puede**:

- Recibir `bytes` o un flujo binario en memoria (sin ruta de fichero).
- Detectar formato, leer el `.ibt` y normalizar.
- Emitir un `TelemetryImport` **completo y consistente**, o un error de
  dominio estructurado (D12). Nunca ambos a la vez, nunca un objeto a medias.
- Depender solo de Pandas, NumPy, biblioteca estándar y la excepción YAML
  documentada en D2.

### El Backend

El Backend **únicamente**:

- orquesta el proceso,
- llama al Core,
- resuelve catálogos y piloto,
- gestiona el almacén de archivos,
- persiste mediante los repositorios ya existentes,
- expone la API.

El Backend **no**:

| Prohibido en el Backend | Motivo |
|-------------------------|--------|
| Interpretar el binario `.ibt` | Eso es el Core |
| Calcular métricas de rendimiento | Eso es análisis (v0.3) |
| Tomar decisiones de análisis / coaching | Eso es el ingeniero (v0.3+) |
| Reimplementar deduplicación lógica del dominio fuera de los repositorios | Usa `content_hash` y `resolve` existentes |
| Inventar columnas o SQL ad hoc | D10: solo repositorios v0.1 |

### Resumen de una línea

| Capa | Verbo | Entrada | Salida |
|------|-------|---------|--------|
| Core | traducir | `bytes` `.ibt` | `TelemetryImport` **o** error estructurado |
| Backend | orquestar y persistir | rutas + `TelemetryImport` | filas SQLite + API |

## 4.2 Vista de capas

```
┌─────────────────────────────────────────────────────────────┐
│  TAURI / FRONTEND                                           │
│  Selección de ficheros · progreso · resultado · errores     │
└────────────────────────────┬────────────────────────────────┘
                             │  HTTP / JSON
┌────────────────────────────▼────────────────────────────────┐
│  BACKEND — orquestación de importación                      │
│  Rutas · hash · almacén · leer bytes · llamar al Core ·     │
│  resolver catálogo · abrir lote · persistir · cerrar lote   │
│  NO interpreta telemetría · NO calcula métricas             │
└───────────────┬─────────────────────────────┬───────────────┘
                │ TelemetryImport             │ SQLite
                ▼                             ▼
┌───────────────────────────────┐   ┌─────────────────────────┐
│  CORE — telemetry/            │   │  backend/db (v0.1)      │
│  bytes → TelemetryImport      │   │  ImportLog              │
│  Sin SQLite, sin HTTP,        │   │  SessionRepository      │
│  sin rutas, sin repositorios  │   │  CatalogRepository      │
└───────────────────────────────┘   │  DriverRepository       │
                                    │  Resolution             │
                                    └─────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│  DISCO — almacén de telemetría gestionado (solo Backend)    │
│  Copia del .ibt · disponibilidad available/archived/missing │
└─────────────────────────────────────────────────────────────┘
```

## 4.3 Dirección de dependencias

```
frontend  →  backend/importing  →  core/telemetry
                 │
                 └──→  backend/db  →  SQLite
```

El Core no importa el backend. El backend no reimplementa lectura de `.ibt`.
El frontend no toca el binario ni calcula huellas de contenido.

## 4.4 Aclaración sobre `DDP_RULES.md` y la telemetría

`DDP_RULES.md` establece que la telemetría no vive en el frontend.
`ARCHITECTURE.md` y `DATA_MODEL.md` precisan el reparto: la **interpretación
del formato** es del Core (`telemetry/parser.py`); la **orquestación,
almacén y persistencia** son del backend. v0.2 sigue esa precisión.

Sobre "abrir el fichero": el modelo exige que ningún otro módulo de dominio
interprete el binario. En v0.2 eso significa:

- el **Backend** obtiene los `bytes` desde el disco (conoce rutas);
- el **Core** es el único que interpreta esos `bytes` como telemetría.

El Core no recibe ni construye rutas de filesystem.

---

# 5. Estructura de módulos propuesta

Nada de esto existe aún como implementación. Es el mapa de destino.

```
core/telemetry/
├── __init__.py
├── parser.py              Frontera pública: detecta formato y delega
├── iracing/
│   ├── __init__.py
│   ├── ibt_reader.py      Lectura binaria del .ibt (header, YAML, buffers)
│   └── map_channels.py    Nombres iRacing → vocabulario interno
├── models.py              TelemetryImport y estructuras anidadas
├── units.py               Conversiones a unidades canónicas del modelo
└── errors.py              Excepciones de dominio del parser

backend/importing/           (nombre fijado en D1)
├── __init__.py
├── pipeline.py            Orquestador: hash → bytes → Core → resolve → persist
├── storage.py             Copia gestionada, rutas, disponibilidad
├── catalog_bind.py        TelemetryImport → IDs de catálogo/piloto
└── service.py             Fachada usable desde endpoints

backend/api/                 (o rutas en main.py al inicio)
└── import_routes.py       Endpoints HTTP de importación
```

Los módulos ya existentes del Core (`analyzer.py`, `consistency.py`, `fuel.py`,
`tires.py`) **permanecen como stubs**. No se implementan en v0.2.

La carpeta `backend/db/` **no se reestructura**. El orquestador la consume.

---

# 6. Contrato de intercambio Core → Backend: `TelemetryImport`

## 6.1 Rol

`TelemetryImport` es el **único objeto de éxito** entre Core y Backend para
una importación completa. El fallo no se modela dentro del objeto: se comunica
como error de dominio estructurado (D12).

| Regla | Detalle |
|-------|---------|
| Salida de éxito del Core | `parser.parse(...)` devuelve exactamente un `TelemetryImport` |
| Salida de fallo del Core | Error de dominio estructurado; **no** se materializa `TelemetryImport` |
| Invariante (D12) | Un `TelemetryImport` es siempre completo y consistente. Nunca parcial |
| Entrada de persistencia del Backend | El orquestador solo persiste si obtuvo un `TelemetryImport` |
| Sin fugas de infraestructura | Ni conexiones SQLite, ni tipos FastAPI, ni rutas de disco, ni IDs locales de BD |
| Naturaleza | Dataclasses / estructuras Python puras. Serializable en tests sin app |

Cualquier dato que el Backend necesite para persistir un `.ibt` debe estar en
este contrato o ser infraestructura pura (ruta de almacén, `content_hash`
calculado por el Backend, piloto activo leído de settings).

## 6.2 Forma lógica

```
TelemetryImport
├── metadata                 Contexto del parseo (ver 6.3)
├── sessions[]               Sesiones observadas en el fichero (ver 6.4)
├── laps[]                   Vueltas, enlazadas a su sesión lógica (ver 6.5)
├── telemetry_files[]        Descriptores de fichero lógico (ver 6.6)
└── warnings[]               Avisos no fatales sobre un resultado ya consistente (ver 6.7)
```

Un `.ibt` de iRacing produce, en el caso normal, **una** sesión, **N** vueltas
y **un** descriptor de telemetría con `scope = session`. El contrato admite
listas para no acoplarse a esa cardinalidad y para absorber fuentes futuras
sin cambiar la frontera. Si esas listas no pueden poblarse de forma
estructuralmente coherente, el Core **no** devuelve el objeto (D12).

## 6.3 `metadata`

| Campo | Contenido |
|-------|-----------|
| `source_format` | `"ibt"` |
| `source_label` | Etiqueta estable, p. ej. `"iracing_ibt"` |
| `tick_rate` | Hz del fichero, si se conoce |
| `record_count` | Número de registros de canal leídos |
| `parsed_at` | Marca temporal del parseo (UTC), informativa |
| `driver_ref` | Identidad de plataforma del piloto del fichero (external_id, nombre si existe) |
| `raw_session_info` | Dict del YAML de iRacing (para `raw_payload` / `attributes`) |

`content_hash` **no** forma parte del contrato del Core: lo calcula el Backend
sobre el fichero en disco (apartado 9.3) y lo usa como clave de
`telemetry_file`.

## 6.4 `sessions[]` (mapeo a `Session`)

Campos que el Core extrae del `.ibt` / session info cuando existen:

| Campo en el contrato | Destino en persistencia | Notas |
|----------------------|-------------------------|-------|
| `session_key` | Enlace lógico interno del contrato | Relaciona laps y telemetry_files antes de existir IDs de BD |
| `started_at` | `session.started_at` | ISO-8601 UTC |
| `session_type` | `session.session_type` | Race / Qualify / Practice / … |
| `sub_session_id` | `session_external_id` (`source=iracing`) | Clave de deduplicación de sesión |
| `track` | resolución → `track_id` | `external_id` y/o nombre |
| `car` | resolución → `car_id` | `external_id` y/o nombre |
| `driver` | resolución → `driver_id` | `external_id` y/o nombre; ver D3 |
| `series` / `season` | resolución opcional | Si faltan, `season_id` nulo |
| `split` | `session.split` | Si aparece |
| `lap_count` | `session.lap_count` | Del sub-header o del conteo de vueltas |
| `best_lap_time_ms` | `session.best_lap_time_ms` | Agregado observado |
| `incidents` | `session.incidents` | Si el YAML lo trae |
| `attributes` | `session.attributes{}` | Weather/conditions y resto M2 |

Campos que un `.ibt` **a menudo no trae** (resultado oficial de carrera):
`start_position`, `finish_position`, `gap_to_winner_ms`, `irating`,
`safety_rating`. En v0.2 quedan nulos / ausentes si no están en el fichero.
No se inventan. La importación de resultados oficiales sin telemetría queda
fuera de alcance (apartado 2.2).

## 6.5 `laps[]`

| Campo | Destino | Notas |
|-------|---------|-------|
| `session_key` | Enlace a `sessions[]` | Obligatorio |
| `lap_number` | `lap.lap_number` | |
| `lap_time_ms` | `lap.lap_time_ms` | Unidades canónicas del modelo (ms) |
| `is_valid` | `lap.is_valid` | Incompleta / outlap / invalid según señales del IBT |
| `context` | `lap.context` | Si se puede deducir (qualify vs race); si no, nulo |

La mejor vuelta de sesión (`is_session_best`) la marca el repositorio existente
al insertar, no el parser.

## 6.6 `telemetry_files[]`

Descriptores lógicos. **Sin rutas de disco** (las pone el Backend al persistir).

| Campo | Destino / uso | Notas |
|-------|---------------|-------|
| `session_key` | Enlace a `sessions[]` | Obligatorio en iRacing |
| `lap_key` | Enlace a una vuelta | Nulo en iRacing (`scope = session`) |
| `format` | `telemetry_file.format` | `".ibt"` |
| `scope` | Deducido al persistir | Siempre `session` para iRacing (M5) |
| `channel_names[]` | `telemetry_file.channels` | Vocabulario interno + no mapeados |
| `attributes` | `telemetry_file.attributes` | Metadatos de fichero no canónicos |

Las **muestras** (series temporales) no viajan en este contrato hacia la
persistencia. El análisis (v0.3) releerá el fichero desde `storage_path`.
Durante el parseo el Core puede materializar DataFrames internamente para
segmentar vueltas y validar canales; eso es detalle de implementación del
Core, no parte del contrato público.

### Vocabulario mínimo de canales

Exigido por las evidencias del modelo (sección 7.10 de `DATA_MODEL.md`):

| Magnitud interna | Origen típico iRacing | Uso futuro (v0.3) |
|------------------|-----------------------|-------------------|
| `brake` | `Brake` | Presión / liberación |
| `throttle` | `Throttle` | Progresión de gas |
| `steering` | `SteeringWheelAngle` | Correcciones |
| `speed` | `Speed` | Ritmo y tracción |
| `gear` | `Gear` | Contexto de conducción |
| `rpm` | `RPM` | Contexto de conducción |
| `lap_dist_pct` | `LapDistPct` | Anclaje a curva / circuito |
| `lat` / `lon` | `Lat` / `Lon` | Trazada (si existen) |

Los canales no mapeados **no se descartan en silencio**: quedan listados en
`channel_names` con su nombre de origen (p. ej. prefijo `iracing:`). Así se
cumple M3 en espíritu: no perder información que hoy no se sabe interpretar.

## 6.7 `warnings[]` (solo sobre éxito completo)

`warnings` describe matices de un `TelemetryImport` **ya consistente**: canal
opcional ausente, campo no canónico faltante, vuelta sin tiempo pero
estructuralmente enlazada, etc. El Backend persiste igual; los avisos viajan
al `result` del lote.

Un warning **nunca** sustituye a un fallo. Si la incoherencia impide un
objeto completo, el Core no emite `TelemetryImport` (D12).

## 6.8 Fallo: error de dominio estructurado (no va dentro del contrato)

Cuando el fichero no puede convertirse de forma consistente, el Core
**devuelve un error estructurado** (excepción de dominio tipada con payload,
o tipo `Result` equivalente en la superficie pública). Ejemplos:

| Error | Cuándo |
|-------|--------|
| `UnsupportedFormatError` | Formato o firma no reconocida |
| `CorruptTelemetryError` | Header incoherente, buffers truncados |
| `MissingSessionInfoError` | No hay YAML de sesión usable |
| `EmptyTelemetryError` | Cero registros de canal |
| `InconsistentTelemetryError` | Datos estructuralmente incoherentes (sesión sin enlace a vueltas, cardinalidad imposible, etc.) |

Nunca códigos HTTP: el Backend las traduce en la capa de API.

**Prohibido:** devolver un `TelemetryImport` con listas a medias, sesiones
huérfanas, o un campo interno de errores mezclado con datos de éxito.

## 6.9 Qué queda fuera del contrato (a propósito)

| Dato | Quién lo aporta |
|------|-----------------|
| Ruta original del usuario | Backend / Tauri |
| `content_hash` | Backend (`storage`) |
| `storage_path` | Backend (`storage`) |
| IDs enteros de SQLite | Backend (repositorios) |
| Piloto activo de la instalación | Backend (`app_setting`) |
| Decisión `kind = self \| external` | Backend al resolver piloto (D3), usando `driver_ref` del contrato |
| Diagnóstico de fallo de parseo | Error de dominio estructurado (D12), no un campo de `TelemetryImport` |

---

# 7. El formato `.ibt` (hechos de partida)

El `.ibt` es el volcado a disco del telemetría en vivo de iRacing (IRSDK).
Estructura lógica:

1. **Header** — versión, tick rate, offsets a session info, variables y buffers.
2. **Disk sub-header** — inicio/fin de sesión, número de vueltas, número de
   registros.
3. **Session info** — documento YAML con circuito, coche, sesión, pilotos,
   condiciones, setup, etc.
4. **Variable headers** — descriptor por canal (nombre, tipo, offset).
5. **Buffers de muestras** — filas a `tickRate` (típicamente 60 Hz).

Reglas de lectura para v0.2:

- Solo canales **escalares** en la primera entrega. Los arrays del IRSDK se
  registran como presentes pero no se expanden hasta que haya un consumidor.
- El YAML completo viaja en `metadata.raw_session_info` y el Backend lo puede
  guardar en `raw_payload` del lote.
- `scope` del `TelemetryFile` resultante es siempre `session` (M5, caso iRacing).

### Dependencias del lector

`ARCHITECTURE.md` limita el Core a Pandas, NumPy y biblioteca estándar.

| Opción | Evaluación |
|--------|------------|
| Biblioteca externa tipo `libibt` | Rápida, pero arrastra PyArrow/Rust al Core y rompe el contrato de dependencias |
| Lector propio con `struct` + YAML | Alineado con el Core; más trabajo; control total |

**Decisión (D2):** lector propio en `core/telemetry/iracing/ibt_reader.py`,
usando `struct` y la biblioteca estándar. Para el YAML de session info se admite
**una** dependencia explícita de parsing YAML (p. ej. PyYAML), documentada como
excepción justificada del parser, o un subconjunto mínimo si el YAML de iRacing
lo permite sin librería. No se introduce `libibt` en el Core.

---

# 8. Flujo de importación (paso a paso)

Una importación de un fichero:

```
 1. Frontend / Tauri entrega ruta(s) absoluta(s) al Backend
 2. Backend valida que el fichero existe y es legible
 3. Backend calcula content_hash (SHA-256 del contenido)
 4. Backend consulta telemetry_file por content_hash
      · si existe y availability=available → resultado "already_imported"
        (no se reescribe; se devuelve la sesión asociada)
 5. Backend copia el fichero al almacén gestionado
 6. Backend lee los bytes del fichero
 7. Backend llama a core.telemetry.parser.parse(bytes)
      · éxito → TelemetryImport completo (D12)
      · fallo → error estructurado; ítem failed; no persistir hechos
 8. Backend abre import_batch (source="iracing_ibt",
      raw_payload desde metadata.raw_session_info)
 9. Dentro de una transacción (SessionRepository.batch):
      a. Resolver simulator = iRacing
      b. Resolver driver (D3: self o external)
      c. Resolver track y car (external_id → natural_key → alias → create)
      d. Resolver season/series si hay datos; si no, nulos
      e. SessionRepository.resolve / create de la sesión
      f. add_laps
      g. register telemetry_file (storage_path, hash, channels, format=".ibt")
      h. métricas solo si el YAML las trae (record_metric, sin pisar)
10. Cerrar lote como completed (o failed si hubo excepción)
11. Responder con resumen: batch_id, session_id, created/matched, laps, hash,
    driver.kind, warnings
```

### Atomicidad

- El lote se abre con `ImportLog.record` / `open`+`close`.
- La escritura de hechos ocurre dentro de `SessionRepository.batch()`.
- Si el parseo falla **antes** de persistir, no hay fila de sesión; el lote
  queda `failed` con el motivo.
- Si la persistencia falla, la transacción revierte hechos; el lote queda
  `failed`.
- La copia en disco en staging se limpia o se marca según política de
  almacenamiento (apartado 9). Un fichero huérfano en disco no corrompe la base.

### Idempotencia

| Caso | Comportamiento |
|------|----------------|
| Mismo fichero (mismo hash) | `already_imported`; no duplica `telemetry_file` ni sesión |
| Mismo `sub_session_id`, otro fichero | Se reconoce la sesión por `external_id`; se puede vincular otro fichero solo si el hash es nuevo y el diseño lo permite — **por defecto en v0.2: un hash nuevo sobre sesión existente enriquece (añade fichero) sin pisar campos ya llenos** |
| Reimportación tras borrado de sesión | El hash ya no está (cascada); se importa de nuevo como alta |

---

# 9. Almacenamiento de ficheros

## 9.1 Ubicación

Directorio gestionado bajo el árbol de datos de la aplicación, no la ruta
original del usuario:

```
database/telemetry/
  iracing/
    ab/
      abcd…hash.ibt
```

Convención propuesta: subcarpeta por prefijo del hash (2 caracteres) + nombre
igual al hash completo + extensión original. La ruta relativa se guarda en
`telemetry_file.storage_path`.

La ruta de importación preferida del usuario (carpeta de telemetrías de
iRacing) puede vivir en `app_setting`; es un atajo de UI, no la fuente de
verdad del fichero ya importado.

## 9.2 Disponibilidad

Se reutilizan los estados ya implementados en v0.1:

| Estado | Uso en v0.2 |
|--------|-------------|
| `available` | Estado tras importación correcta |
| `archived` | Reservado; política concreta aplazada (`ROADMAP` / decisión 4 de BD) |
| `missing` | Detectable en verificación si el fichero desaparece |

v0.2 **no** implementa archivado automático. Solo deja el mecanismo listo.

## 9.3 Huella

- Algoritmo: **SHA-256** del contenido completo del fichero.
- Se calcula en el Backend **antes** de parsear, sobre el fichero de origen.
- Es la clave natural de `telemetry_file` (ya en el esquema).

---

# 10. Resolución de catálogo y piloto

El orquestador no inventa un sistema nuevo: usa `Resolution` y los
repositorios de v0.1, en el orden ya fijado por el diseño de BD:

1. `external_id` de plataforma (`source = "iracing"`)
2. Clave natural (nombre canónico, etc.)
3. Alias
4. Creación

### Reglas específicas v0.2

| Entidad | Comportamiento |
|---------|----------------|
| `simulator` | Debe existir "iRacing". Si no está, se crea una vez como parte del seed/resolución inicial del pipeline |
| `track` | Resolver por `track_id` de iRacing; si no, por nombre / alias ("Bathurst" → Mount Panorama) |
| `car` | Resolver por id de iRacing; si no, por nombre / alias |
| `driver` | Ver apartado 10.1 (D3) |
| `series` / `season` | Mejor esfuerzo; pueden quedar nulos sin tumbar la importación |
| `corner` | No se crea desde el `.ibt` en v0.2. Las curvas del catálogo siguen viniendo del seed/catálogo; el análisis (v0.3) las usará vía `LapDistPct` + perfil de circuito |

Política de escritura: **enriquecer, no corregir**. Si el catálogo ya tiene
nombre y el YAML trae otro, no se pisa. Los huecos sí se rellenan.

## 10.1 Telemetría de otros pilotos (D3)

### Justificación respecto a `DATA_MODEL.md` v1.0

El modelo ya contempla este caso. `Driver.kind` admite `self` · `external` y
está documentado así:

> `kind` distingue al piloto propio de los pilotos externos cuyas vueltas se
> usan como referencia. Es el mecanismo que permite almacenar datos de terceros
> sin entidades nuevas.

La sección 11 (Garage61) y las vueltas de referencia / fantasma confirman que
el dominio **debe** poder alojar telemetría de terceros. Rechazar un `.ibt` solo
porque el cliente iRacing no es el piloto activo contradiría ese diseño: el
modelo ya resolvió el problema con `kind`, no con un muro de importación.

### Comportamiento en v0.2

| Situación | Acción |
|-----------|--------|
| `driver_ref` coincide con el piloto activo (`kind = self`) | Resolver a ese `driver_id`; importar con normalidad |
| `driver_ref` es otro cliente iRacing | Resolver o crear `Driver` con `kind = external` y su `external_id`; importar igualmente; conservar sesión, vueltas y fichero |
| Ya existe un `Driver` external con ese `external_id` | Reutilizarlo (no duplicar) |
| Incompatibilidad técnica (corrupto, formato inválido, incoherencia estructural) | El Core emite error estructurado; el Backend rechaza ese fichero (D12) |

No se exige flag de confirmación. No se atribuye en silencio la sesión al
piloto `self`. El Backend elige `self` vs `external` al resolver; el Core solo
informa `driver_ref` en el contrato.

### Uso posterior

Las sesiones y vueltas del piloto `external` quedan disponibles como hechos.
En v0.3+ podrán usarse como vuelta de referencia sin reimportar ni cambiar el
esquema.

---

# 11. Superficie HTTP y UI

## 11.1 API mínima

| Método | Ruta | Función |
|--------|------|---------|
| `POST` | `/imports/ibt` | Importar uno o varios paths locales |
| `GET` | `/imports/{batch_id}` | Estado y resultado del lote |
| `GET` | `/imports` | Listado reciente de lotes |
| `GET` | `/sessions/{id}` | Lectura de sesión ya existente (si no está) |
| `GET` | `/sessions/{id}/telemetry` | Metadatos del fichero (no las muestras) |

El cuerpo de importación trabaja con **rutas locales** (aplicación de
escritorio), no con upload multipart de ficheros enormes. Tauri resuelve la
selección de fichero y entrega la ruta al Backend.

Respuesta de importación (forma lógica):

```
{
  batch_id,
  status,                    # completed | failed | partial (multi-file)
  items: [
    {
      path,
      content_hash,
      outcome,               # created | already_imported | enriched | failed
      session_id?,
      lap_count?,
      driver: { id, kind },   # self | external
      matched: { track, car, driver },
      warnings?,
      error?
    }
  ]
}
```

## 11.2 Frontend mínimo

La página `Telemetry` / `Sessions` existente se conecta al pipeline:

1. Botón o zona de selección de `.ibt` (diálogo nativo vía Tauri).
2. Lista de ficheros en cola con estado por ítem.
3. Resumen al terminar: sesiones nuevas, ya importadas, externas, errores.
4. Enlace a la sesión creada.

Sin visualización de canales en v0.2 (eso consume análisis o un visor que
releerá el fichero). Mostrar metadatos basta: circuito, coche, vueltas, hash,
fecha, si el piloto es propio o externo.

---

# 12. Decisiones de diseño de v0.2

Decisiones que este documento **toma** y que, tras la congelación, quedan fijas
para la implementación.

| ID | Decisión | Elección |
|----|----------|----------|
| **D1** | Paquete de orquestación en backend | `backend/importing/` — evita chocar con la palabra reservada `import` de Python |
| **D2** | Lector `.ibt` | Propio en el Core; sin `libibt`; YAML con dependencia mínima documentada |
| **D3** | Fichero de otro cliente iRacing | Importar siempre que el fichero sea técnicamente válido. Si no es el piloto activo, crear/resolver `Driver` con `kind = external`. Solo se rechaza por error técnico |
| **D4** | Unidad de lote | Un `import_batch` por petición de importación (1..N ficheros). Cada fichero es un ítem del `result` |
| **D5** | `raw_payload` del lote | Session info YAML (o su dict) de cada fichero, indexado por hash/path. No se vuelcan las muestras |
| **D6** | Muestras en BD | Nunca. Solo metadatos + path + hash + lista de canales |
| **D7** | Reimportación del mismo hash | No-op exitoso (`already_imported`), no error |
| **D8** | Análisis al importar | No se dispara. v0.2 termina en hechos persistidos |
| **D9** | Formatos en v0.2 | Solo `.ibt`. `.csv` / `.rpy` quedan reconocibles como "aún no soportados" con error claro |
| **D10** | Persistencia | Solo a través de repositorios v0.1; cero SQL nuevo en el orquestador |
| **D11** | Contrato Core → Backend | `TelemetryImport` es el único objeto de éxito del Core y la única entrada de dominio que el orquestador persiste |
| **D12** | Atomicidad del resultado del Core | El Core nunca devuelve un `TelemetryImport` parcialmente válido. Éxito = objeto completo y consistente. Fallo = error de dominio estructurado. Sin mezclar éxito y fallo en el mismo valor |

---

# 13. Decisiones abiertas (no bloquean el diseño)

| ID | Tema | Notas | Cuándo decidir |
|----|------|-------|----------------|
| A1 | Política de archivado de telemetría | Ya es la decisión pendiente 4 de `DATABASE_DESIGN.md` | Antes de que el volumen duela; no bloquea v0.2 |
| A2 | Borrado de sesión desde la UI de importación | El esquema ya permite cascada; la UI puede aplazarse | Fase de UI o inmediatamente después |
| A3 | Importación de resultados sin `.ibt` | Fuera de alcance; conviene diseñar después de ver huecos reales del YAML | Post-v0.2 |
| A4 | Watch folder (carpeta vigilada) | Atajo de producto; no necesario para el pipeline | Post-v0.2 |
| A5 | Seed inicial del catálogo iRacing | ¿Cuántos circuitos/coches sembrar vs crear lazy al importar? | Fase 4; lazy es válido para empezar |

Las decisiones pendientes 1–3 de `DATABASE_DESIGN.md` (UUID estable entre
instalaciones, granularidad del registro, alcance del borrado) **no se
reabren** aquí. v0.2 vive con las recomendaciones ya adoptadas en v0.1.

---

# 14. Orden de implementación

Misma metodología que v0.1: una fase → implementación → verificación →
revisión → aprobación. **No se avanza de fase sin aprobación.**

Cada fase se valida con un criterio falsable. Si no lo cumple, no está
terminada.

| # | Fase | Entrega | Verificación |
|:-:|------|---------|--------------|
| **0** | Contrato `TelemetryImport` | `models.py`, `errors.py`, superficie de `parser.parse(bytes) -> TelemetryImport` (éxito) o error estructurado (fallo); invariante D12 | Estructuras importables en tests; Core sin SQLite/HTTP/rutas/repositorios; no existe campo `errors` en el contrato de éxito |
| **1** | Lector IBT | `ibt_reader.py` lee header, sub-header, YAML y canales escalares desde bytes | Sobre un `.ibt` de fixture: tick rate, `sub_session_id` / track / car presentes, N registros > 0, lista de canales no vacía |
| **2** | Normalización | Vocabulario interno, unidades canónicas, segmentación en vueltas → `TelemetryImport` completo | `brake`/`throttle`/`steering`/`speed` en `channel_names`; vueltas con `lap_time_ms` en ms; `telemetry_files[0].scope` lógico = session; basura/corrupto → error estructurado, nunca objeto parcial |
| **3** | Almacén y huella | `storage.py` + SHA-256 + layout en `database/telemetry/` | Mismo fichero → mismo hash; copia bajo el almacén; segunda copia no duplica bytes si ya existe el hash |
| **4** | Orquestador | `pipeline.py` + `catalog_bind.py` usando repositorios v0.1 y solo `TelemetryImport` | Fixture propio: 1 batch completed, 1 session, N laps, 1 telemetry_file; reimportar → `already_imported`; fixture de otro cliente → `Driver.kind = external` sin rechazar; "Pole" no se almacena si apareciera |
| **5** | API HTTP | Rutas de importación y consulta de lotes | `POST /imports/ibt` con path de fixture devuelve resumen; `GET` del lote coincide con la BD |
| **6** | UI mínima | Selección de fichero + feedback de resultado | Desde la app: elegir fixture → ver sesión creada / ya importada / externa / error legible |
| **7** | Verificación de etapa | Script `verify_v0_2.py` (o equivalente) + regresión v0.1 intacta | Pipeline E2E verde; `verify_db.py` / fases 0–10 de v0.1 siguen verdes; ningún documento congelado tocado |

### Por qué este orden

- **0 → 2** construyen el Core de dentro afuera, testeable sin FastAPI ni UI.
- **3 → 4** añaden el adaptador de persistencia sin abrir aún la red.
- **5 → 6** exponen el valor al usuario solo cuando el pipeline ya es correcto.
- **7** cierra la etapa con la misma disciplina de regresión que cerró v0.1.

### Fixtures

v0.2 necesita al menos un `.ibt` real (o recorte legítimo) bajo
`assets/telemetry/fixtures/`, más un manifiesto JSON con los valores
esperados (track id, car id, lap_count, canales obligatorios). Sin fixture no
hay fase 1 aprobable.

Los fixtures **no se inventan en código**: se obtienen de una sesión real del
piloto o de un volcado de prueba controlado. El manifiesto sí se redacta a mano
como oráculo de verificación.

---

# 15. Criterios de cierre de v0.2

v0.2 se considera **completa** cuando:

1. Este documento está aprobado y congelado.
2. Las fases 0–7 están implementadas, verificadas y aprobadas una a una.
3. Un `.ibt` de fixture se importa de extremo a extremo desde la UI o la API.
4. La reimportación del mismo fichero no duplica hechos.
5. Un `.ibt` de otro cliente queda persistido bajo `Driver.kind = external`.
6. La capa `backend/db` no ha cambiado, salvo fixes demostrables documentados.
7. Los cuatro documentos congelados de v0.1 permanecen idénticos.
8. La regresión de infraestructura v0.1 sigue verde.
9. `CHANGELOG.md` registra v0.2 como etapa cerrada.

---

# 16. Relación con el roadmap

| Punto del `ROADMAP.md` | Efecto de v0.2 |
|------------------------|----------------|
| §1 Versionado del catálogo | Sigue aplazado; v0.2 puede crear filas lazy y alimentará el problema real |
| §2 Prioridad por origen | Sigue aplazado; solo hay `source=iracing_ibt` |
| §3 Trazabilidad del motor | No aplica; v0.2 no calcula derivados |
| §4 Presupuesto de índices | No se toca; la importación usará los índices ya creados en v0.1 |

Cuando v0.2 esté cerrada, el siguiente candidato natural es **v0.3 — Core de
análisis**, que parte de `Session` + `TelemetryFile` ya persistidos.

---

# 17. Reglas de uso de este documento

1. La implementación de v0.2 se deriva de este documento, y este documento se
   deriva de los cuatro documentos congelados de v0.1. La cadena no se
   recorre en sentido contrario.
2. Una fase no puede introducir un módulo o responsabilidad que contradiga los
   apartados 4–6 (frontera Core/Backend y contrato `TelemetryImport`).
3. Si hace falta un atributo de dominio nuevo, se detiene el trabajo y se abre
   v1.1 del modelo o del diseño de BD. No se altera el esquema en silencio.
4. **Ninguna modificación estructural del diseño durante la implementación.**
   Ante una necesidad nueva, el desarrollo se detiene y se revisa este
   documento con re-aprobación explícita.
5. Ninguna fase se da por terminada sin cumplir su verificación.
6. Este documento no tiene autoridad para modificar documentos congelados.

---

# 18. Cierre del diseño

## Estado del documento

| Aspecto | Situación |
|---------|-----------|
| Base | `ARCHITECTURE.md` · `DATA_MODEL.md` v1.0 · `DATABASE_DESIGN.md` v1.0 · `DDP_RULES.md`, sin modificar |
| Contrato Core → Backend | `TelemetryImport` (metadata · sessions · laps · telemetry_files · warnings) |
| Invariante de éxito | D12: objeto completo o error estructurado; nunca parcial |
| Frontera de capas | Explícita en el apartado 4 |
| Decisiones cerradas | D1–D12 |
| Decisiones abiertas | A1–A5, ninguna bloqueante |
| Implementación | Completa — fases 0–7 verificadas |

**Versión 1.0 · congelada · referencia oficial de la etapa v0.2 (cerrada).**

---

*Documento congelado. La implementación se deriva de aquí; no al revés.*
