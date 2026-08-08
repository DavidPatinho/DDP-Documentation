# DDP v0.3 — Motor de análisis (Core)

> Documento de diseño de la etapa v0.3.
> Se deriva de `ARCHITECTURE.md`, `DATA_MODEL.md`, `DATABASE_DESIGN.md`,
> `DDP_RULES.md` e `IMPORT_PIPELINE.md`, todos **congelados**. Este documento
> no los modifica.
>
> Ante cualquier conflicto con un documento congelado, el documento congelado
> tiene prioridad y este diseño se corrige.

---

## Estado

| Aspecto | Situación |
|---------|-----------|
| Versión | v0.3 |
| Estado | **Congelado — diseño aprobado** |
| Esquema SQLite | Sin cambios. Se usa la capa de persistencia de v0.1 tal cual |
| Congelado | Sí. 08/08/2026 |
| Implementación | En curso (Fase 0) |

**Versión 1.0 · congelada · referencia oficial para la implementación del
motor de análisis (v0.3).**

Ningún cambio estructural durante la implementación. Si aparece una necesidad
arquitectónica real, se detiene el desarrollo y se abre una revisión de este
documento — la misma regla que cerró `IMPORT_PIPELINE.md` en v0.2.
---

# 1. Objetivo

Diseñar e implementar el **motor de análisis** que convierte hechos ya
persistidos (`Session` · `Lap` · `TelemetryFile`) en una medición objetiva
regenerable (`Analysis` + hallazgos por curva):

```
Session + Lap[] + bytes de TelemetryFile (+ referencia opcional)
    → SessionAnalysis (Core)
    → resolución de curvas / vuelta de referencia (Backend)
    → Analysis · analysis_corner_finding persistidos
```

Al cerrar v0.3, DDP debe poder:

1. Analizar una sesión importada a partir de su telemetría disponible.
2. Comparar contra una vuelta de referencia (`own` · `ideal` · `external`).
3. Calcular la vuelta teórica ideal y métricas de consistencia.
4. Mapear pérdida / ganancia de tiempo por curva.
5. Resumir entradas del piloto y errores detectables sin opinar.
6. Persistir el resultado con trazabilidad (`engine_version`, `computed_at`,
   `inputs_hash`).
7. Regenerar el análisis sin tocar hechos ni el esquema.
8. Convivir resultados de versiones distintas del motor para la misma sesión.
9. Exponer el resultado por API y mostrarlo de forma mínima en la UI.

Lo que **no** hace v0.3: interpretar, priorizar insights, redactar informes,
proponer objetivos, calcular Driver Index, actualizar `TrackProfile` /
`Career`, ni actuar como ingeniero de pista. Eso es **v0.4 — Ingeniero IA**.

---

# 2. Alcance

## 2.1 Dentro de v0.3

| Pieza | Descripción |
|-------|-------------|
| Contrato `SessionAnalysis` | Única salida pública de éxito del Core hacia el Backend para un análisis |
| Lectura de telemetría para análisis | Releer bytes del `.ibt` ya importado; reutilizar el lector de v0.2 |
| `telemetry/analyzer.py` | Medición: deltas, ideal teórica, pérdida por curva, entradas, errores |
| `telemetry/consistency.py` | Índices de regularidad de la sesión |
| `telemetry/fuel.py` / `tires.py` | Solo si hay canales suficientes; resultado en `attributes{}` (M2) |
| Anclaje a curvas | `lap_dist_pct` + geometría de catálogo vía `Corner.attributes{}` |
| Orquestación de análisis | Backend como adaptador: carga hechos, llama al Core, resuelve IDs, persiste |
| Política de regeneración | Vaciar / sustituir derivados de `analysis` sin tocar hechos |
| Versionado del motor | `engine_version` + `inputs_hash` |
| API de análisis | Endpoints mínimos para disparar, consultar y regenerar |
| UI mínima | Estado de análisis, mapa de pérdida por curva, métricas clave |
| Verificación | Suite por fase + regresión v0.1 y v0.2 intactas |

## 2.2 Fuera de v0.3

| Pieza | Motivo | Etapa prevista |
|-------|--------|----------------|
| `engineer/coach.py` · insights priorizados | Interpreta; no mide | v0.4 |
| `engineer/recommendations.py` | Plan de entrenamiento | v0.4 |
| `reports/*` · `AnalysisReport` | Juicio narrativo y documento leído | v0.4 |
| `scoring/driver_score.py` · `DriverIndex` | Depende de decisiones pendientes 1–3 del modelo | v0.4 o posterior |
| `TrackProfile` · `Career` | Estado acumulado / interpretación de largo plazo | v0.4 o posterior |
| `objectives/*` · progreso de objetivos | Ciclo de intención + coaching | v0.4 |
| Sectores oficiales (`Sector`) | Entidad futura; las fuentes trabajan por curva | Cuando exista la entidad |
| `algorithm_version` · `parameters_hash` | Roadmap §3; un solo algoritmo en v0.3 | Cuando haya varios algoritmos |
| Fuentes adicionales de importación | Roadmap §5; no es análisis | Posterior a v0.2 |
| Visualización rica de canales | Consumidor de UI; no es el motor | Mejora post-v0.3 |

## 2.3 Frontera con etapas cerradas

La infraestructura (v0.1) y el pipeline de importación (v0.2) quedan
**cerrados**:

- No se modifican `ARCHITECTURE.md`, `DATA_MODEL.md`, `DATABASE_DESIGN.md`,
  `DDP_RULES.md`, `IMPORT_PIPELINE.md`.
- No se modifican migraciones ni el esquema SQLite.
- No se modifica la capa `backend/db/*` salvo **correcciones de errores
  demostrables** (fallo reproducible + prueba que lo pincha).
- El motor de v0.3 **consume** `AnalysisRepository`, `SessionRepository` y el
  lector de telemetría de v0.2 tal como existen.
- El parser de importación (`TelemetryImport`) **no se reabre**. El análisis
  tiene su propio contrato de salida.

Si al diseñar o implementar aparece un atributo de dominio que no cabe en el
modelo, se detiene el trabajo y se abre una versión 1.1 del documento
correspondiente. No se improvisa una columna.

---

# 3. Principios que gobiernan el diseño

Heredados de los documentos congelados; aquí solo se aplican al motor.

| # | Principio | Consecuencia en v0.3 |
|:-:|-----------|----------------------|
| P1 | Frontera Core / Backend estricta | Apartado 4. El Core mide; el Backend orquesta y persiste |
| P2 | Analysis mide, no opina | Nada de texto pedagógico, insights ni recomendaciones en el contrato |
| P3 | Hechos inmutables | El análisis nunca reescribe `Session`, `Lap` ni `TelemetryFile` |
| P4 | Derivados regenerables | `analysis` es desechable: se puede vaciar y reconstruir con IDs nuevos |
| P5 | Un dato, un lugar | No se duplican tiempos de vuelta ya persistidos como hechos |
| P6 | La curva es la unidad de trabajo | Pérdida/ganancia se expresa por `Corner`, no por sector |
| P7 | Trazabilidad obligatoria | Toda fila derivada lleva `engine_version` · `computed_at` · `inputs_hash` |
| P8 | El fichero no entra en SQLite | El análisis releerá bytes; no materializa muestras en BD |
| P9 | Dependencias hacia dentro | El Core no conoce FastAPI, SQLite, rutas ni repositorios |
| P10 | Una versión de motor, un resultado por sesión | Clave natural `(session_id, engine_version)` |

---

# 4. Frontera Core / Backend

Esta separación es la regla de oro de v0.3. Si una responsabilidad cae del
lado equivocado, el diseño está mal aplicado.

## 4.1 El Core

El Core **únicamente** transforma:

```
AnalysisRequest (estructuras Python puras + bytes de telemetría)
        ↓
SessionAnalysis
```

El Core **no conoce**:

| Prohibido en el Core | Motivo |
|----------------------|--------|
| SQLite / conexiones / SQL | La persistencia es del Backend |
| FastAPI / HTTP / status codes | El transporte es del Backend |
| Repositorios (`AnalysisRepository`, …) | Son adaptadores de infraestructura |
| Rutas de disco / `storage_path` / `app_setting` | El Backend gestiona el sistema de archivos |
| IDs enteros locales de SQLite | Identidades de persistencia, no de dominio de cálculo |
| GitHub / red / APIs externas | El Core no tiene I/O de entorno |
| Piloto activo, preferencias, tema | Contexto de instalación |
| Redacción de coaching / informes | Eso es v0.4 |

El Core **sí puede**:

- Recibir `bytes` de telemetría y hechos de sesión/vueltas como estructuras
  puras.
- Reutilizar el lector `.ibt` de v0.2 **solo como biblioteca de lectura**
  (sin pasar por `TelemetryImport` ni por persistencia).
- Emitir un `SessionAnalysis` **completo y consistente**, o un error de
  dominio estructurado. Nunca ambos a la vez, nunca un objeto a medias.
- Depender solo de Pandas, NumPy, biblioteca estándar y la excepción YAML
  ya admitida en el parser (D2 de v0.2).

## 4.2 El Backend

El Backend **únicamente**:

- decide *cuándo* analizar (disparo manual, post-import opcional futuro, regeneración),
- carga hechos y bytes desde SQLite / almacén,
- construye el `AnalysisRequest`,
- llama al Core,
- resuelve identidades de persistencia (vuelta de referencia, curvas),
- calcula o valida `inputs_hash` según el apartado 8,
- persiste mediante `AnalysisRepository`,
- expone la API.

El Backend **no**:

| Prohibido en el Backend | Motivo |
|-------------------------|--------|
| Calcular deltas, ideal, pérdidas o consistencia | Eso es el Core |
| Interpretar canales de telemetría | Eso es el Core |
| Inventar columnas o SQL ad hoc | Solo repositorios v0.1 |
| Escribir hechos al “corregir” un análisis | Los hechos no se tocan |
| Generar texto de ingeniero | Eso es v0.4 |

## 4.3 Resumen de una línea

| Capa | Verbo | Entrada | Salida |
|------|-------|---------|--------|
| Core | medir | `AnalysisRequest` | `SessionAnalysis` **o** error estructurado |
| Backend | orquestar y persistir | `session_id` + hechos/bytes | filas `analysis` + API |

## 4.4 Vista de capas

```
┌─────────────────────────────────────────────────────────────┐
│  TAURI / FRONTEND                                           │
│  Disparo de análisis · estado · mapa por curva · métricas   │
└────────────────────────────┬────────────────────────────────┘
                             │  HTTP / JSON
┌────────────────────────────▼────────────────────────────────┐
│  BACKEND — orquestación de análisis                         │
│  Cargar Session/Lap/TelemetryFile · leer bytes ·            │
│  construir AnalysisRequest · llamar al Core ·               │
│  resolver corner/lap IDs · persistir AnalysisRepository     │
│  NO calcula métricas · NO interpreta                        │
└───────────────┬─────────────────────────────┬───────────────┘
                │ SessionAnalysis             │ SQLite
                ▼                             ▼
┌───────────────────────────────┐   ┌─────────────────────────┐
│  CORE — telemetry/            │   │  backend/db (v0.1)      │
│  AnalysisRequest →            │   │  AnalysisRepository     │
│  SessionAnalysis              │   │  SessionRepository      │
│  Sin SQLite, sin HTTP,        │   │  CatalogRepository      │
│  sin rutas, sin repositorios  │   │                         │
└───────────────────────────────┘   └─────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│  DISCO — almacén de telemetría (solo Backend lee bytes)     │
└─────────────────────────────────────────────────────────────┘
```

## 4.5 Dirección de dependencias

```
frontend  →  backend/analysis  →  core/telemetry
                 │
                 └──→  backend/db  →  SQLite
```

El Core no importa el backend. El backend no reimplementa medición. El
frontend no toca telemetría ni calcula deltas.

---

# 5. Entradas y salidas del motor

## 5.1 Entradas (`AnalysisRequest`)

Estructura pura construida por el Backend. **Sin IDs de SQLite.**

```
AnalysisRequest
├── session                  Hechos de la sesión a analizar
│   ├── session_key          Clave lógica del contrato
│   ├── started_at
│   ├── session_type
│   ├── lap_count
│   ├── best_lap_time_ms
│   └── attributes{}         Condiciones / contexto no canónico
├── laps[]                   Vueltas ya persistidas (hechos)
│   ├── lap_key              Clave lógica
│   ├── lap_number
│   ├── lap_time_ms
│   ├── is_valid
│   └── context?
├── telemetry                Bytes + metadatos del fichero
│   ├── format               ".ibt" en v0.3
│   ├── content_hash         Informativo; no sustituye inputs_hash
│   ├── channel_names[]
│   └── bytes                Contenido binario en memoria
├── track_geometry           Geometría mínima para anclar curvas
│   ├── track_ref            Identidad de catálogo (nombre / external_id)
│   └── corners[]            name · aliases · start_pct · end_pct · category?
├── reference                Política y material de comparación
│   ├── policy               own | ideal | external
│   ├── own_lap_key?         Si policy=own y se fuerza una vuelta concreta
│   └── external_lap?        Serie / hechos de una vuelta externa (si aplica)
└── options                  Parámetros no configurables aún (reserva)
    └── include_fuel_tires   bool; best-effort si hay canales
```

### Origen de cada entrada

| Entrada | Quién la aporta |
|---------|-----------------|
| `session` · `laps[]` | Backend desde `SessionRepository` |
| `telemetry.bytes` | Backend desde almacén (`storage_path`) |
| `track_geometry` | Backend desde catálogo (`Corner` + `attributes{}`) |
| `reference` | Backend según política del apartado 7.3 |
| Cálculo | Solo el Core |

### Disponibilidad del fichero

Si `telemetry_file.availability != available`, el Backend **no llama al Core**.
Devuelve un error de orquestación claro (`TelemetryUnavailableError` o
equivalente HTTP). Sin bytes no hay análisis reconstruible
(`DATABASE_DESIGN.md` §7.3–7.4).

## 5.2 Salida de éxito (`SessionAnalysis`)

Único objeto de éxito del Core. Completo o no existe (misma disciplina que
D12 de v0.2).

## 5.3 Salida de fallo

Error de dominio estructurado. Nunca un `SessionAnalysis` parcial. Nunca
códigos HTTP desde el Core.

---

# 6. Contrato público del motor: `SessionAnalysis`

## 6.1 Rol

| Regla | Detalle |
|-------|---------|
| Salida de éxito del Core | `analyze(...)` devuelve exactamente un `SessionAnalysis` |
| Salida de fallo del Core | Error de dominio estructurado; **no** se materializa el objeto |
| Invariante | Un `SessionAnalysis` es siempre completo y consistente. Nunca parcial |
| Entrada de persistencia del Backend | El orquestador solo persiste si obtuvo un `SessionAnalysis` |
| Sin fugas de infraestructura | Ni conexiones SQLite, ni tipos FastAPI, ni rutas, ni IDs locales |
| Naturaleza | Dataclasses / estructuras Python puras |

## 6.2 Forma lógica

```
SessionAnalysis
├── meta
│   ├── engine_version
│   ├── computed_at          UTC ISO-8601 (informativo; el Backend puede
│   │                        fijar el de persistencia al escribir)
│   ├── inputs_hash          Huella canónica de las entradas del cálculo
│   └── status               p. ej. "complete"
├── reference
│   ├── source               own | ideal | external
│   ├── lap_key?             Clave lógica de la vuelta usada (si aplica)
│   └── note?                Aviso no fatal (p. ej. ideal sintética)
├── theoretical_best_ms
├── consistency_metrics{}    Salida de consistency.py
├── driver_inputs_summary{}  Agregados de freno / gas / volante
├── errors_detected[]        Observaciones objetivas sin coaching
├── corner_findings[]        Pérdida/ganancia por curva (ver 6.3)
├── unattributed_gains[]     Ganancias sin curva concreta (categoría)
├── attributes{}             fuel/tires/estrategia best-effort (M2)
└── warnings[]               Matices sobre un resultado ya consistente
```

## 6.3 `corner_findings[]`

| Campo | Destino | Notas |
|-------|---------|-------|
| `corner_ref` | resolución → `corner_id` | Nombre canónico o alias; **nunca** un entero de BD |
| `time_loss_ms` | `analysis_corner_finding.time_loss_ms` | Puede ser negativo (más rápido que la referencia) |
| `estimated_gain_ms` | `analysis_corner_finding.estimated_gain_ms` | ≥ 0 o nulo |

Si una pérdida no puede anclarse a una curva del catálogo, **no** se inventa
una curva: o bien queda fuera del mapa (warning), o bien se expresa como
`unattributed_gains` si es una categoría ("curvas lentas").

## 6.4 Qué queda fuera del contrato (a propósito)

| Dato | Quién lo aporta |
|------|-----------------|
| `session_id` entero | Backend |
| `reference_lap_id` entero | Backend al resolver `lap_key` |
| `corner_id` entero | Backend al resolver `corner_ref` |
| Rutas de disco | Backend |
| Texto de insight / recomendación | v0.4 |
| `DriverIndex` / informe | v0.4 |
| Diagnóstico de fallo | Error estructurado, no un campo del éxito |

## 6.5 Errores de dominio (fallo)

| Error | Cuándo |
|-------|--------|
| `UnsupportedTelemetryError` | Formato no analizable |
| `CorruptTelemetryError` | Bytes ilegibles / buffers truncados |
| `InsufficientTelemetryError` | Canales mínimos ausentes (`lap_dist_pct`, tiempos, …) |
| `InsufficientLapsError` | No hay vueltas válidas para medir |
| `MissingGeometryError` | No hay geometría de curvas usable para el circuito |
| `InvalidReferenceError` | La política de referencia no puede aplicarse |
| `InconsistentAnalysisError` | Resultado estructuralmente incoherente |

Un warning **nunca** sustituye a un fallo.

## 6.6 Trazabilidad de cada hallazgo (D13)

Todo hallazgo emitido por el motor —en particular cada entrada de
`corner_findings[]` y de `unattributed_gains[]`— debe poder rastrearse hasta
los hechos y la configuración que lo originaron:

| Origen | Dónde queda anclado |
|--------|---------------------|
| Vueltas usadas | `AnalysisRequest.laps[]` (cubiertas por `inputs_hash`) |
| Referencia de comparación | `SessionAnalysis.reference` (`source` · `lap_key`) |
| Algoritmo | En v0.3, identificado por `engine_version` (un solo algoritmo por entrega) |
| Versión del motor | `SessionAnalysis.meta.engine_version` |

Un hallazgo no lleva huella propia: hereda la trazabilidad del
`SessionAnalysis` que lo contiene. Si no puede anclarse a esos orígenes de
forma coherente, el Core **no** emite `SessionAnalysis` (D12).

## 6.7 Determinismo del motor (D14)

El motor es **determinista**:

```
mismas entradas (AnalysisRequest canónico)
  + misma engine_version
  ⇒ mismo SessionAnalysis
```

Misma medición, mismos hallazgos, mismo `inputs_hash`, mismos enteros en ms.
La única excepción admitida es `computed_at`, marca temporal de ejecución
(informativa); no forma parte del contrato de igualdad del resultado.

---

# 7. Estrategia de análisis

## 7.1 Cadena interna (solo medición)

```
bytes + hechos
      │
      ▼
 releer / alinear canales por lap_dist_pct
      │
      ▼
 analyzer.py                 mide ritmo, deltas, ideal, entradas, errores
      │
      ├─> consistency.py     regularidad de la tanda
      ├─> fuel.py            best-effort → attributes{}
      └─> tires.py           best-effort → attributes{}
      │
      ▼
 mapa por curva              corner_findings[] + unattributed_gains[]
      │
      ▼
 SessionAnalysis             contrato público
```

Ninguna etapa interpreta ni redacta. Ninguna etapa salta hacia `engineer/` o
`reports/`.

## 7.2 Canales mínimos

Heredados del vocabulario de v0.2. Para un análisis completo se exigen al
menos:

| Canal | Uso |
|-------|-----|
| `lap_dist_pct` | Anclaje espacial a curva |
| `speed` | Ritmo y pérdidas |
| `brake` · `throttle` · `steering` | `driver_inputs_summary` y errores |
| Tiempos de vuelta (hechos `Lap`) | Base temporal; no se recalculan como hechos |

Si faltan canales opcionales (combustible, neumáticos), el análisis sigue
siendo completo; fuel/tires quedan ausentes en `attributes{}` y se emite
warning.

## 7.2bis Política de selección de vueltas (`ddp-lap-selection-1.0`)

Antes de medir consistencia / ritmo, el Core clasifica cada vuelta del
request con la política `ddp-lap-selection-1.0` (`lap_selection.py`).

| Conjunto | Quién entra | Uso |
|----------|-------------|-----|
| `reference_pool` | Estructuralmente elegibles | Mejor vuelta (`own`), pool de referencia |
| `ideal_pool` | Estructuralmente elegibles | Ideal teórica / mejores tramos |
| `consistency` | Elegibles menos `pace_outlier` | `consistency_metrics` (ritmo / stdev) |

**Estructuralmente elegible:** `is_valid` y `lap_time_ms > 0`.

| Motivo de exclusión | Objetivo | Afecta |
|---------------------|----------|--------|
| `incomplete` | `is_valid = false` | Todos los conjuntos |
| `no_positive_time` | sin tiempo cerrado positivo | Todos los conjuntos |
| `pace_outlier` | `lap_time_ms > best_ms × 1.02` (≥ 4 elegibles) | Solo `consistency` |

La exclusión `pace_outlier` **no** se etiqueta como calentamiento: sin
evidencia objetiva de temperatura de neumático u out-lap (p. ej. OnPitRoad),
el motor no afirma la causa. Tampoco se excluye automáticamente la primera
vuelta por número.

El informe completo viaja en `SessionAnalysis.attributes.lap_selection` para
que el Backend/UI muestren vueltas usadas, excluidas y el motivo.

## 7.3 Política de referencia

`Analysis.reference_source` ya está cerrado en esquema: `own` · `ideal` ·
`external`.

| Política | Significado en v0.3 | Selección |
|----------|---------------------|-----------|
| `own` | Mejor vuelta válida de la propia sesión (o `own_lap_key` forzada) | Por defecto |
| `ideal` | Vuelta teórica por mejores tramos de la sesión | Cuando se pida explícitamente o no haya vuelta propia usable |
| `external` | Vuelta de un `Driver.kind = external` aportada en el request | Solo si el Backend incluye `external_lap` |

Reglas:

1. El Core **no elige política de producto** por sí solo más allá de lo que
   traiga `reference.policy`. Si la política pedida es imposible, falla con
   `InvalidReferenceError` (no inventa otra en silencio).
2. El Backend puede ofrecer un modo "auto" en la API que traduzca a una de
   las tres políticas **antes** de llamar al Core. Esa traducción es
   orquestación, no medición.
3. Comparar con externos es posible gracias a v0.2 (`Driver.kind = external`);
   v0.3 solo consume esa telemetría ya importada.

## 7.4 Anclaje a curvas sin tocar el modelo

`Corner` no tiene atributos de distancia en el modelo v1.0 (están ausentes a
propósito). La geometría usable vive en el mecanismo ya previsto:

> Geometría del circuito → `Track.attributes{}` · `Corner.attributes{}`
> (`DATA_MODEL.md`, extensibilidad M2).

**Decisión de v0.3 (D5):** el Backend proyecta, desde el catálogo, un
`track_geometry` con `start_pct` / `end_pct` por curva leídos de
`Corner.attributes{}` (o equivalentes documentados). El Core solo consume ese
descriptor.

Si el circuito no tiene geometría cargada, el análisis **falla** con
`MissingGeometryError` en v0.3 (no se inventan curvas ni se “adivinan”
cortes). Sembrar geometría de circuitos reales es trabajo de catálogo /
fixtures, no un cambio de esquema.

## 7.5 Qué significa “medir, no opinar” en la práctica

| Permitido en `SessionAnalysis` | Prohibido (v0.4) |
|--------------------------------|------------------|
| `time_loss_ms = 180` en Forrest's Elbow | "Hay que abrir gas antes en Forrest's Elbow" |
| `errors_detected = ["lockup_detected"]` | "Trabaja la sensibilidad de freno" |
| `consistency_metrics.stdev_ms = 240` | "Consistencia sobresaliente" |
| `unattributed_gains = [{category: "slow_corners", ms: 250}]` | "El próximo objetivo es Forest Elbow" |

Los textos de `errors_detected` son **etiquetas técnicas estables**, no
narrativa al piloto.

## 7.6 Fuel y tires

Módulos presentes en la estructura del Core. En v0.3:

- Se implementan solo si los canales del fixture lo permiten.
- Su salida viaja en `SessionAnalysis.attributes{}` (`fuel_summary`,
  `tires_summary`, …), tal como fija el modelo hasta que existan modelos
  propios.
- Su ausencia **no** tumba el análisis.

---

# 8. Versionado del motor

## 8.1 Dos ejes independientes

| Eje | Qué cambia | Cómo se resuelve |
|-----|-----------|------------------|
| Versión de esquema SQLite | Forma de las tablas | Migración (cerrada en v0.1) |
| `engine_version` | Fórmula del Core | Recálculo selectivo |

No se confunden. Mejorar el cálculo **no** es una migración.

## 8.2 Formato de `engine_version`

Cadena estable y comparable, p. ej. `ddp-analysis-0.3.0`.

Reglas:

1. La emite el Core (constante de entrega del motor de análisis).
2. El Backend la persiste tal cual; no la reescribe.
3. Misma versión + mismos inputs ⇒ mismo `inputs_hash` y mismo resultado
   medible (reproducibilidad).
4. Al subir la versión, los análisis antiguos **permanecen** hasta regenerarse.
   La clave `(session_id, engine_version)` permite convivencia.

## 8.3 `inputs_hash`

Huella de las **entradas del cálculo**, no de la configuración futura.

Incluye, de forma canónica (orden estable, UTF-8, sin rutas):

- hechos de sesión/vueltas usados,
- `content_hash` del fichero de telemetría,
- geometría de curvas suministrada,
- política de referencia y material de referencia,
- `engine_version`.

**Decisión (D6):** el Core calcula `inputs_hash` sobre el `AnalysisRequest`
canónico y lo devuelve en `SessionAnalysis.meta`. El Backend lo persiste. Así
la reproducibilidad es verificable sin el Backend.

## 8.4 Relación con el Roadmap §3

`algorithm_version` y `parameters_hash` quedan **fuera de v0.3**. Con un solo
algoritmo y sin parámetros configurables, `engine_version` + `inputs_hash`
bastan, tal como aplaza `ROADMAP.md` §3. No se pide cambio de esquema.

---

# 9. Política de regeneración de derivados

Heredada de `DATABASE_DESIGN.md` §2.4 y §7; aquí solo se operacionaliza para
v0.3.

## 9.1 Grado de `analysis`

`analysis` es **desechable**:

- se puede vaciar y reconstruir con identificadores nuevos,
- nada fuera de la capa derivada la referencia,
- la política de escritura del repositorio es **sustituir**, no enriquecer.

## 9.2 Disparadores

| Disparador | Comportamiento |
|------------|----------------|
| Análisis pedido sobre sesión sin fila de la versión actual | Calcular y `store` |
| Reanálisis con la misma `engine_version` | Sustituir fila + hallazgos |
| Nueva `engine_version` | Insertar nueva fila; conservar la antigua |
| Regeneración masiva | Vaciar por versión obsoleta (`clear`) y recalcular en lote |
| Fichero `missing` / `archived` | No regenerar; conservar último análisis si existía; señalar indisponibilidad |
| Borrado de sesión | Cascada ya definida en v0.1; no hay acción extra |

## 9.3 Qué nunca se regenera desde el motor

- `Session`, `Lap`, `TelemetryFile`, métricas de sesión
- Catálogos
- Cualquier entidad de v0.4 (`AnalysisReport`, `DriverIndex`, …)

## 9.4 Orden futuro (fuera de v0.3, para no romper el grafo)

Cuando existan capas posteriores:

```
analysis → driver_index → analysis_report
```

Regenerar `analysis` invalidará lógicamente lo que cuelga. v0.3 no implementa
esa cascada porque esas tablas aún no las produce el producto; solo deja el
contrato de medición listo.

## 9.5 Atomicidad

Persistir un análisis ocurre dentro de la transacción del repositorio
(`store` borra e inserta hallazgos de la misma versión). Un fallo deja la
versión anterior intacta o ausente; nunca un análisis a medias.

---

# 10. Organización interna de módulos

Nada de esto existe aún como implementación completa. Es el mapa de destino.
Los stubs actuales se convierten en módulos reales **solo** en la medida del
alcance de v0.3.

```
core/telemetry/
├── parser.py                 (v0.2 — cerrado; reutilizable como lector)
├── iracing/                  (v0.2 — cerrado)
├── models.py                 TelemetryImport (v0.2) — no se reabre
├── analysis_models.py        AnalysisRequest · SessionAnalysis · errores
├── analyzer.py               Orquestador interno de medición + frontera pública
├── consistency.py            Regularidad
├── fuel.py                   Best-effort → attributes
├── tires.py                  Best-effort → attributes
├── reference.py              Construcción de serie de referencia / ideal
├── corners.py                Recorte por lap_dist_pct → findings
├── inputs.py                 Resúmenes de freno/gas/volante
├── errors_detect.py          Errores objetivos (lockup, etc.)
└── hashing.py                Canonicalización + inputs_hash

backend/analysis/             (nombre del paquete de orquestación)
├── __init__.py
├── pipeline.py               Orquestador: load → request → Core → resolve → store
├── geometry.py               Corner.attributes{} → track_geometry
├── reference_policy.py       Traducción API auto → policy del request
└── service.py                Fachada para endpoints

backend/api/
└── analysis_routes.py        Endpoints HTTP de análisis
```

### Qué permanece stub en v0.3

```
core/engineer/*               v0.4
core/reports/*                v0.4
core/scoring/*                v0.4
core/objectives/*             v0.4
```

La carpeta `backend/db/` **no se reestructura**. El orquestador la consume.

---

# 11. Superficie HTTP y UI

## 11.1 API mínima

| Método | Ruta | Función |
|--------|------|---------|
| `POST` | `/sessions/{id}/analysis` | Analizar o reanalizar la sesión |
| `GET` | `/sessions/{id}/analysis` | Leer análisis (versión actual o `?engine_version=`) |
| `GET` | `/sessions/{id}/analysis/findings` | Hallazgos por curva |
| `POST` | `/analysis/regenerate` | Lote: regenerar sesiones con versión obsoleta / faltante |
| `GET` | `/analysis/engine` | `engine_version` vigente del Core |

Cuerpo lógico de `POST /sessions/{id}/analysis`:

```
{
  reference_policy: "own" | "ideal" | "external" | "auto",
  external_lap_id?: number,     // solo Backend; se proyecta al request
  force?: boolean               // reanalizar aunque inputs_hash coincida
}
```

Respuesta lógica:

```
{
  session_id,
  analysis_id,
  engine_version,
  inputs_hash,
  status,
  reference: { source, lap_id? },
  theoretical_best_ms,
  consistency_metrics,
  findings: [{ corner_id, corner_name, time_loss_ms, estimated_gain_ms }],
  unattributed_gains,
  warnings,
  outcome: "created" | "replaced" | "unchanged" | "failed"
}
```

`outcome = unchanged` cuando `force` es falso y ya existe análisis de la misma
versión con el mismo `inputs_hash`.

## 11.2 Frontend mínimo

En la vista de sesión / telemetría:

1. Botón “Analizar” / “Reanalizar”.
2. Estado: sin análisis · calculando · completo · fichero no disponible.
3. Métricas clave: ideal teórica, consistencia, referencia usada.
4. Tabla o lista de pérdida/ganancia por curva.
5. Etiquetas técnicas de errores detectados (sin narrativa de coach).

Sin informes, sin insights prioritarios, sin Driver Index.

---

# 12. Decisiones de diseño de v0.3

Decisiones que este documento **toma** y que, tras la congelación, quedan fijas
para la implementación.

| ID | Decisión | Elección |
|----|----------|----------|
| **D1** | Alcance de v0.3 | Solo medición → `Analysis`. Interpretación e informes = v0.4 |
| **D2** | Contrato Core → Backend | `SessionAnalysis` es el único objeto de éxito; fallo = error estructurado |
| **D3** | Paquete de orquestación | `backend/analysis/` |
| **D4** | Frontera de IDs | El Core usa claves lógicas (`lap_key`, `corner_ref`); el Backend resuelve enteros |
| **D5** | Geometría de curvas | `Corner.attributes{}` / proyección `track_geometry`; sin cambio de modelo |
| **D6** | `inputs_hash` | Lo calcula el Core sobre el request canónico |
| **D7** | Referencia por defecto | `own` (mejor vuelta válida); `auto` es traducción del Backend |
| **D8** | Persistencia | Solo `AnalysisRepository` v0.1; política sustituir |
| **D9** | Fuel/tires | Best-effort en `attributes{}`; no bloquean |
| **D10** | Post-import automático | No obligatorio en v0.3; el disparo mínimo es explícito por API/UI |
| **D11** | Reproducibilidad | Misma versión + mismos inputs ⇒ mismo hash y mismas métricas enteras |
| **D12** | Atomicidad del resultado del Core | Éxito completo o error; nunca parcial |
| **D13** | Trazabilidad de hallazgos | Todo hallazgo es trazable a vueltas, referencia, algoritmo y `engine_version` vía el `SessionAnalysis` padre (apartado 6.6) |
| **D14** | Determinismo | Mismas entradas + misma `engine_version` ⇒ mismo `SessionAnalysis` (igualdad salvo `computed_at`; apartado 6.7) |

---

# 13. Decisiones abiertas (no bloquean el diseño)

| ID | Tema | Notas | Cuándo decidir |
|----|------|-------|----------------|
| A1 | Semilla de geometría por circuito | Qué circuitos traen `start_pct`/`end_pct` de fábrica | Fase 2 / fixtures |
| A2 | Catálogo de etiquetas de `errors_detected` | Lista cerrada inicial vs abierta | Fase 3 |
| A3 | Análisis automático tras importar | Conveniencia de producto; no cambia el motor | Post-v0.3 o fase UI |
| A4 | Profundidad de fuel/tires en v0.3 | Stub verificado vs implementación parcial | Fase 3 según canales del fixture |
| A5 | Comparación externa multi-vuelta | Una vuelta basta para cerrar `external` | Cuando haya UX de selección |

Las decisiones pendientes 1–3 de `DATA_MODEL.md` (fórmula del índice,
dimensiones, error score) **no se reabren** aquí: pertenecen a scoring /
v0.4+.

---

# 14. Fases de implementación

Misma metodología que v0.1 y v0.2: una fase → implementación → verificación →
revisión → aprobación. **No se avanza de fase sin aprobación.**

Cada fase se valida con un criterio falsable. Si no lo cumple, no está
terminada.

| # | Fase | Entrega | Verificación |
|:-:|------|---------|--------------|
| **0** | Contrato `SessionAnalysis` | `analysis_models.py`, errores, superficie `analyze(request) -> SessionAnalysis`; invariante D12 | Estructuras importables; Core sin SQLite/HTTP/rutas/repositorios/IDs enteros; no existe campo de errores dentro del éxito |
| **1** | Relectura y alineación | Reutilizar lector v0.2; alinear canales por `lap_dist_pct` y vueltas | Fixture: series por vuelta no vacías; canales mínimos presentes; basura → error estructurado |
| **2** | Geometría y recorte por curva | `corners.py` + proyección Backend `geometry.py` | Dado un track_geometry de fixture, cada muestra cae en ≤1 curva; hallazgo con `corner_ref` resoluble |
| **3** | Medición core | `analyzer.py` + `reference.py` + `consistency.py` + inputs/errors | Ideal teórica > 0; `time_loss_ms` coherente con referencia; consistency presente; sin texto de coaching en la salida |
| **4** | Hash y versión | `hashing.py` + `engine_version` estable | Mismo request ⇒ mismo `inputs_hash`; cambiar una vuelta cambia el hash; versión no vacía |
| **5** | Orquestador | `backend/analysis/pipeline.py` + resolve + `AnalysisRepository.store` | Sesión fixture → 1 `analysis` + N findings; reanalizar misma versión sustituye; versión nueva convive; sin fichero → no persiste |
| **6** | API HTTP | Rutas de análisis / lectura / regeneración | `POST` crea; `GET` coincide con BD; `unchanged` si hash idéntico; regresión import v0.2 verde |
| **7** | UI mínima | Disparo + mapa por curva + estado | Desde la app: analizar fixture → ver pérdidas por curva / error legible si falta fichero |
| **8** | Verificación de etapa | Script `verify_v0_3.py` + regresión v0.1/v0.2 | E2E verde; documentos congelados intactos; `CHANGELOG.md` listo para cerrar la etapa |

### Por qué este orden

- **0 → 4** construyen el Core de dentro afuera, testeable sin FastAPI ni UI.
- **5** añade el adaptador de persistencia sin abrir aún la red.
- **6 → 7** exponen el valor al usuario solo cuando la medición ya es correcta.
- **8** cierra la etapa con la misma disciplina de regresión que cerró v0.2.

### Fixtures

v0.3 reutiliza el `.ibt` de v0.2 y añade:

1. Manifiesto de geometría del circuito del fixture (`start_pct` / `end_pct`
   por curva).
2. Oráculo de métricas esperadas (tolerancias documentadas para tiempos
   enteros en ms).

Sin geometría de fixture la fase 2 no es aprobable.

---

# 15. Criterios de cierre de v0.3

v0.3 se considera **completa** cuando:

1. Este documento está aprobado y congelado.
2. Las fases 0–8 están implementadas, verificadas y aprobadas una a una.
3. Una sesión importada se analiza de extremo a extremo desde la UI o la API.
4. El resultado persiste en `analysis` + `analysis_corner_finding` con
   trazabilidad completa.
5. Reanalizar con la misma versión sustituye; con versión nueva convive.
6. Regenerar con fichero ausente no corrompe hechos ni inventa mediciones.
7. La salida del Core no contiene coaching, informes ni scoring.
8. La capa `backend/db` no ha cambiado, salvo fixes demostrables documentados.
9. Los documentos congelados de v0.1 y v0.2 permanecen idénticos.
10. La regresión v0.1 y v0.2 sigue verde.
11. `CHANGELOG.md` registra v0.3 como etapa cerrada.

---

# 16. Relación con el roadmap y etapas vecinas

| Documento / punto | Efecto de v0.3 |
|-------------------|----------------|
| `IMPORT_PIPELINE.md` | Queda cerrado; el análisis releerá ficheros ya importados |
| Roadmap §3 Trazabilidad fina | Sigue aplazado; v0.3 usa `engine_version` + `inputs_hash` |
| Roadmap §1–2 Catálogo / prioridad de fuentes | No aplica al motor de medición |
| README · v0.4 Ingeniero IA | Consume `SessionAnalysis` / `Analysis` ya persistido |

---

# 17. Necesidades que tocarían documentos congelados

Durante este diseño **no** se ha encontrado ninguna necesidad que obligue a
modificar un documento congelado.

Puntos vigilados y resolución **sin** abrir v1.1:

| Tensión | Resolución dentro de lo congelado |
|---------|-----------------------------------|
| `Corner` sin distancias canónicas | Geometría vía `Corner.attributes{}` (M2) + proyección `track_geometry` |
| Sectores en stubs antiguos del Core | Fuera de alcance; unidad = curva |
| Trazabilidad `algorithm_version` | Aplazada (Roadmap §3); no se pide columna nueva |
| Fuel/tires sin entidad propia | `Analysis.attributes{}` como ya fija el modelo |

Si en revisión o implementación aparece una necesidad real de cambiar un
congelado, **se detiene el trabajo**, se justifica y se espera aprobación
explícita antes de continuar.

---

# 18. Reglas de uso de este documento

1. La implementación de v0.3 se deriva de este documento, y este documento se
   deriva de los documentos congelados. La cadena no se recorre al revés.
2. Una fase no puede introducir una responsabilidad que contradiga los
   apartados 4–6 (frontera y contrato `SessionAnalysis`).
3. Si hace falta un atributo de dominio nuevo, se detiene el trabajo y se abre
   v1.1 del modelo o del diseño de BD.
4. **Ninguna modificación estructural del diseño durante la implementación.**
   Ante una necesidad nueva, el desarrollo se detiene y se revisa este
   documento con re-aprobación explícita.
5. Ninguna fase se da por terminada sin cumplir su verificación.
6. Este documento no tiene autoridad para modificar documentos congelados.

---

# 19. Cierre del diseño

## Estado del documento

| Aspecto | Situación |
|---------|-----------|
| Base | Documentos congelados de v0.1 + `IMPORT_PIPELINE.md` v1.0, sin modificar |
| Contrato Core → Backend | `SessionAnalysis` |
| Invariante de éxito | D12: objeto completo o error estructurado; nunca parcial |
| Trazabilidad de hallazgos | D13 |
| Determinismo | D14 |
| Frontera de capas | Explícita en el apartado 4 |
| Decisiones cerradas | D1–D14 |
| Decisiones abiertas | A1–A5, ninguna bloqueante |
| Implementación | En curso — se deriva de este documento |

**Versión 1.0 · congelada · referencia oficial de la etapa v0.3.**

---

*Documento congelado. La implementación se deriva de aquí; no al revés.*
