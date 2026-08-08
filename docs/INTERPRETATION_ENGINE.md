# DDP v0.4 — Motor de interpretación (Core)

> Documento de diseño de la etapa v0.4.
> Se deriva de `ARCHITECTURE.md`, `DATA_MODEL.md`, `DATABASE_DESIGN.md`,
> `DDP_RULES.md`, `IMPORT_PIPELINE.md` y `ANALYSIS_ENGINE.md`, todos
> **congelados**. Este documento no los modifica.
>
> Ante cualquier conflicto con un documento congelado, el documento congelado
> tiene prioridad y este diseño se corrige.

---

## Estado

| Aspecto | Situación |
|---------|-----------|
| Versión | v0.4 |
| Estado | **Congelado — diseño aprobado** |
| Esquema SQLite | Sin cambios. Se usa la capa de persistencia de v0.1 tal cual |
| Congelado | Sí. 08/08/2026 |
| Implementación | En curso (Fase 7) |

**Versión 1.0 · congelada · referencia oficial para la implementación del
motor de interpretación (v0.4).**

Ningún cambio estructural durante la implementación. Si aparece una necesidad
arquitectónica real, se detiene el desarrollo y se abre una revisión de este
documento — la misma regla que cerró `ANALYSIS_ENGINE.md` en v0.3.

---

# 1. Objetivo

Diseñar e implementar el **motor de interpretación** que convierte una medición
objetiva ya existente (`SessionAnalysis` / `Analysis`) en conocimiento útil
para el piloto: diagnóstico priorizado, fortalezas y debilidades, plan de
mejora y un informe de sesión regenerable:

```
SessionAnalysis (ya medido por v0.3)
    → SessionInterpretation (Core)
    → resolución de identidades / número de informe (Backend)
    → AnalysisReport (scope = session) + hijas persistidos
```

Al cerrar v0.4, DDP debe poder responder, **sin volver a medir telemetría**:

1. ¿Cuál es el principal problema del piloto en la sesión?
2. ¿Qué curva merece mayor atención?
3. ¿Qué patrón se repite durante la sesión?
4. ¿Qué fortalezas aparecen?
5. ¿Qué debilidades aparecen?
6. ¿Qué prioridad de entrenamiento debería tener?
7. ¿Qué recomendaciones objetivas pueden emitirse?
8. ¿Qué plan de mejora puede proponerse?

Y, además:

9. Persistir el resultado como `AnalysisReport` (`scope = session`) con
   trazabilidad (`engine_version`, `computed_at`, `inputs_hash`).
10. Regenerar la interpretación en el sitio sin tocar hechos ni el esquema.
11. Exponer el resultado por API y mostrarlo de forma mínima en la UI.

Lo que **no** hace v0.4: importar ficheros, releer `.ibt`, recalcular deltas,
ideal teórica, pérdida por curva ni consistencia. Eso permanece exclusivamente
en **v0.3 — Analysis Engine**.

---

# 2. Alcance

## 2.1 Dentro de v0.4

| Pieza | Descripción |
|-------|-------------|
| Contrato `SessionInterpretation` | Única salida pública de éxito del Core hacia el Backend para una interpretación de sesión |
| Entrada `InterpretationRequest` | Proyección pura de un `SessionAnalysis` ya existente (+ contexto opcional) |
| `engineer/coach.py` | Diagnóstico: problema principal, curva prioritaria, patrones, insights (1–3) |
| `engineer/recommendations.py` | Acciones de entrenamiento con criterio de éxito medible |
| `reports/session.py` | Composición del informe de sesión a partir del diagnóstico |
| Orquestación de interpretación | Backend como adaptador: carga `Analysis`, llama al Core, resuelve IDs, regenera informe |
| Política de regeneración | Actualizar `analysis_report` **en el sitio** (identidad conservada) |
| Versionado del motor | `engine_version` propia de interpretación + `inputs_hash` |
| API de interpretación | Endpoints mínimos para disparar, consultar y regenerar |
| UI mínima | Estado del informe, insights, fortalezas/mejoras, recomendaciones |
| Verificación | Suite por fase + regresión v0.1 / v0.2 / v0.3 intactas |

## 2.2 Fuera de v0.4

| Pieza | Motivo | Etapa prevista |
|-------|--------|----------------|
| Relectura de telemetría / `analyze(...)` | Medición; frontera cerrada en v0.3 | Cerrado |
| `scoring/driver_score.py` · `DriverIndex` | Decisiones pendientes 1–3 de `DATA_MODEL.md` §17 sin cerrar | Posterior a v0.4 (o revisión dedicada) |
| `reports/weekly.py` · informe semanal | Agregación multi-sesión; no responde al alcance de sesión | Posterior |
| `reports/career.py` · `Career` | Perspectiva de largo plazo | Posterior |
| Actualización de `TrackProfile` | Estado acumulado por combo; requiere política multi-sesión | Posterior |
| Persistencia de `Objective` | La definición es permanente y autorizada por persona (`DATABASE_DESIGN.md` §2.2 / §3.2). v0.4 solo **propone** acciones en `recommendations[]` | Cuando exista flujo de aceptación |
| `objective_progress` | Depende de objetivos aceptados | Con objetivos |
| Modelos generativos opacos (LLM) | Rompen trazabilidad determinista exigida por el principio fundamental | Fuera de esta etapa |
| `algorithm_version` · `parameters_hash` | Roadmap §3; un solo juego de reglas en v0.4 | Cuando haya varios |
| Sectores oficiales (`Sector`) | Entidad futura | Cuando exista la entidad |
| Visualización rica / PDF | Consumidor de UI; no es el motor | Mejora post-v0.4 |

## 2.3 Frontera con etapas cerradas

Las etapas v0.1, v0.2 y v0.3 quedan **cerradas**:

- No se modifican `ARCHITECTURE.md`, `DATA_MODEL.md`, `DATABASE_DESIGN.md`,
  `DDP_RULES.md`, `IMPORT_PIPELINE.md`, `ANALYSIS_ENGINE.md`.
- No se modifican migraciones ni el esquema SQLite.
- No se modifica la capa `backend/db/*` salvo **correcciones de errores
  demostrables** (fallo reproducible + prueba que lo pincha).
- El motor de v0.4 **consume** `AnalysisRepository`,
  `AnalysisReportRepository` y el contrato `SessionAnalysis` tal como existen.
- El motor de análisis (`telemetry/analyzer.py` y hermanos) **no se reabre**.
  La interpretación tiene su propio contrato de entrada y salida.

Si al diseñar o implementar aparece un atributo de dominio que no cabe en el
modelo, se detiene el trabajo y se abre una versión 1.1 del documento
correspondiente. No se improvisa una columna.

---

# 3. Principios que gobiernan el diseño

Heredados de los documentos congelados; aquí solo se aplican al motor de
interpretación.

| # | Principio | Consecuencia en v0.4 |
|:-:|-----------|----------------------|
| P1 | Frontera Core / Backend estricta | Apartado 4. El Core interpreta; el Backend orquesta y persiste |
| P2 | Analysis mide, Interpretation opina con evidencia | Nada de deltas ni relectura de canales en v0.4 |
| P3 | Hechos inmutables | La interpretación nunca reescribe `Session`, `Lap`, `TelemetryFile` ni `Analysis` |
| P4 | Derivados regenerables | `analysis_report` se regenera **en el sitio** (identidad conservada) |
| P5 | Un dato, un lugar | No se vuelven a almacenar `time_loss_ms` ni métricas de `Analysis` como hechos nuevos |
| P6 | La curva es la unidad de trabajo | Prioridades y recomendaciones se anclan a `Corner` cuando la evidencia lo permite |
| P7 | Trazabilidad obligatoria | Toda conclusión cita mediciones de v0.3 + versiones de ambos motores |
| P8 | Dependencias hacia dentro | El Core no conoce FastAPI, SQLite, rutas ni repositorios |
| P9 | La IA simplifica | 1–3 insights; formato Insight / Evidencia / Acción (`DDP_RULES.md`) |
| P10 | Sin reglas opacas | Toda regla de interpretación es explícita, versionada y verificable |
| P11 | Determinismo | Mismas entradas + misma `engine_version` ⇒ misma interpretación (salvo `computed_at`) |

### Principio fundamental de v0.4

> Toda recomendación emitida por el motor de interpretación debe poder
> justificarse exclusivamente a partir de las mediciones objetivas generadas
> por la v0.3. El motor nunca inventa conclusiones ni aplica reglas opacas.

Cadena de trazabilidad obligatoria:

```
Telemetría
    ↓
v0.3 (medición objetiva)
    ↓
SessionAnalysis / Analysis
    ↓
v0.4 (interpretación)
    ↓
Informe / Recomendación / Plan de mejora
```

La v0.4 interpreta únicamente resultados ya medidos. **Nunca vuelve a medir.**

---

# 4. Frontera Core / Backend

Esta separación es la regla de oro de v0.4. Si una responsabilidad cae del
lado equivocado, el diseño está mal aplicado.

## 4.1 El Core

El Core **únicamente** transforma:

```
InterpretationRequest (estructuras Python puras; sin bytes de telemetría)
        ↓
SessionInterpretation
```

El Core **no conoce**:

| Prohibido en el Core | Motivo |
|----------------------|--------|
| SQLite / conexiones / SQL | La persistencia es del Backend |
| FastAPI / HTTP / status codes | El transporte es del Backend |
| Repositorios | Son adaptadores de infraestructura |
| Rutas de disco / `storage_path` / `app_setting` | El Backend gestiona el entorno |
| IDs enteros locales de SQLite | Identidades de persistencia, no de dominio de juicio |
| Bytes `.ibt` / canales / `lap_dist_pct` | Eso es medición (v0.3) |
| `analyze(...)` / módulos de medición | Frontera cerrada |
| GitHub / red / APIs externas / LLM | Sin I/O de entorno ni juicio opaco |
| Piloto activo, preferencias, tema | Contexto de instalación |

El Core **sí puede**:

- Recibir un `SessionAnalysis` (o su proyección canónica) como estructura pura.
- Recibir contexto opcional ya materializado (objetivos activos, etiqueta de
  combo) **sin** releer telemetría.
- Emitir un `SessionInterpretation` **completo y consistente**, o un error de
  dominio estructurado. Nunca ambos a la vez, nunca un objeto a medias.
- Depender solo de la biblioteca estándar de Python y, si hiciera falta para
  agregaciones triviales, NumPy — nunca de Pandas sobre series de telemetría.

## 4.2 El Backend

El Backend **únicamente**:

- decide *cuándo* interpretar (disparo manual, post-análisis opcional futuro,
  regeneración),
- carga el `Analysis` persistido (y hechos mínimos de sesión/piloto/catálogo),
- construye el `InterpretationRequest`,
- llama al Core,
- resuelve identidades de persistencia (`corner_id`, `report_number`,
  `session_id`, `driver_id`),
- calcula o valida `inputs_hash` según el apartado 8,
- persiste mediante `AnalysisReportRepository` (**actualizar en el sitio**),
- expone la API.

El Backend **no**:

| Prohibido en el Backend | Motivo |
|-------------------------|--------|
| Priorizar insights o redactar coaching | Eso es el Core |
| Recalcular pérdidas, ideal o consistencia | Eso es v0.3 |
| Releer o parsear `.ibt` | Eso es v0.2 / v0.3 |
| Inventar columnas o SQL ad hoc | Solo repositorios v0.1 |
| Escribir hechos al “corregir” un informe | Los hechos no se tocan |
| Borrar un informe para recrearlo con otro `id` | Política §2.4 de BD |

## 4.3 Resumen de una línea

| Capa | Verbo | Entrada | Salida |
|------|-------|---------|--------|
| Core | interpretar | `InterpretationRequest` | `SessionInterpretation` **o** error estructurado |
| Backend | orquestar y persistir | `session_id` + `Analysis` | filas `analysis_report` + API |

## 4.4 Vista de capas

```
┌─────────────────────────────────────────────────────────────┐
│  TAURI / FRONTEND                                           │
│  Disparo de interpretación · insights · plan · informe      │
└────────────────────────────┬────────────────────────────────┘
                             │  HTTP / JSON
┌────────────────────────────▼────────────────────────────────┐
│  BACKEND — orquestación de interpretación                   │
│  Cargar Analysis · proyectar InterpretationRequest ·        │
│  llamar al Core · resolver corner/report IDs ·              │
│  regenerar AnalysisReportRepository (en el sitio)           │
│  NO mide telemetría · NO inventa juicio                     │
└───────────────┬─────────────────────────────┬───────────────┘
                │ SessionInterpretation       │ SQLite
                ▼                             ▼
┌───────────────────────────────┐   ┌─────────────────────────┐
│  CORE — engineer/ + reports/  │   │  backend/db (v0.1)      │
│  InterpretationRequest →      │   │  AnalysisRepository     │
│  SessionInterpretation        │   │  AnalysisReportRepo     │
│  Sin SQLite, sin HTTP,        │   │  SessionRepository      │
│  sin rutas, sin repositorios, │   │                         │
│  sin bytes de telemetría      │   │                         │
└───────────────────────────────┘   └─────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│  v0.3 Analysis (ya persistido) — única fuente de medición   │
└─────────────────────────────────────────────────────────────┘
```

## 4.5 Dirección de dependencias

```
frontend  →  backend/interpretation  →  core/engineer + core/reports
                 │
                 ├──→  backend/db  →  SQLite
                 └──→  (lectura) Analysis ya producido por v0.3
```

El Core de interpretación no importa el backend. El backend no reimplementa
juicio. El frontend no prioriza insights. Ningún módulo de v0.4 importa el
orquestador de análisis para recalcular: solo **lee** resultados.

---

# 5. Entradas y salidas del motor

## 5.1 Entradas (`InterpretationRequest`)

Estructura pura construida por el Backend. **Sin IDs de SQLite. Sin bytes de
telemetría.**

```
InterpretationRequest
├── analysis                     Medición ya completa (proyección de SessionAnalysis)
│   ├── meta
│   │   ├── engine_version       Versión del motor de análisis (v0.3)
│   │   ├── inputs_hash          Huella de las entradas de medición
│   │   └── status               Debe ser "complete"
│   ├── reference
│   │   ├── source               own | ideal | external
│   │   ├── lap_key?
│   │   └── note?
│   ├── theoretical_best_ms?
│   ├── consistency_metrics{}
│   ├── driver_inputs_summary{}
│   ├── errors_detected[]        Etiquetas técnicas estables de v0.3
│   ├── corner_findings[]
│   │   ├── corner_ref           Clave lógica; nunca un entero de BD
│   │   ├── time_loss_ms?
│   │   └── estimated_gain_ms?
│   ├── unattributed_gains[]
│   ├── attributes{}             fuel/tires u otros best-effort de v0.3
│   └── warnings[]               Solo informativos; no sustituyen fallo
├── session_context              Hechos mínimos para redactar sin medir
│   ├── session_key
│   ├── started_at
│   ├── session_type?
│   ├── track_ref                Identidad de catálogo (nombre / external_id)
│   ├── car_ref?
│   └── best_lap_time_ms?
└── options
    ├── max_insights             Entero; por defecto 3; techo duro = 3
    └── locale                   Reserva; v0.4 emite textos en el idioma del
                                 producto ya usado en fuentes (sin i18n completa)
```

### Origen de cada entrada

| Entrada | Quién la aporta |
|---------|-----------------|
| `analysis.*` | Backend desde `AnalysisRepository` (+ hallazgos), proyectado al contrato de v0.3 |
| `session_context` | Backend desde `SessionRepository` + catálogo |
| Cálculo / juicio | Solo el Core de interpretación |

### Disponibilidad del análisis

Si no existe un `Analysis` completo para la sesión (o la versión pedida), el
Backend **no llama al Core**. Devuelve un error de orquestación claro
(`AnalysisUnavailableError` o equivalente HTTP). Sin medición no hay
interpretación reconstruible.

El Backend **no** dispara automáticamente un reanálisis de v0.3 para “arreglar”
la ausencia. Medir e interpretar son operaciones distintas y explícitas.

### Qué queda explícitamente fuera del request

| Dato | Motivo |
|------|--------|
| `telemetry.bytes` / canales | Medición |
| Series temporales | Medición |
| `storage_path` | Infraestructura |
| IDs enteros (`analysis_id`, `corner_id`, …) | Persistencia |
| Histórico multi-sesión / `TrackProfile` / `Career` | Fuera de alcance de v0.4 |
| `DriverIndex` | Decisiones pendientes de fórmula |

## 5.2 Salida de éxito (`SessionInterpretation`)

Único objeto de éxito del Core. Completo o no existe (misma disciplina que D12
de v0.2 / v0.3).

## 5.3 Salida de fallo

Error de dominio estructurado. Nunca un `SessionInterpretation` parcial. Nunca
códigos HTTP desde el Core.

---

# 6. Contrato público del motor: `SessionInterpretation`

## 6.1 Rol

| Regla | Detalle |
|-------|---------|
| Salida de éxito del Core | `interpret(...)` devuelve exactamente un `SessionInterpretation` |
| Salida de fallo del Core | Error de dominio estructurado; **no** se materializa el objeto |
| Invariante | Un `SessionInterpretation` es siempre completo y consistente. Nunca parcial |
| Entrada de persistencia del Backend | El orquestador solo persiste si obtuvo un `SessionInterpretation` |
| Sin fugas de infraestructura | Ni conexiones SQLite, ni tipos FastAPI, ni rutas, ni IDs locales |
| Naturaleza | Dataclasses / estructuras Python puras |

## 6.2 Forma lógica

```
SessionInterpretation
├── meta
│   ├── engine_version           Versión del motor de interpretación (v0.4)
│   ├── computed_at              UTC ISO-8601 (informativo)
│   ├── inputs_hash              Huella canónica del InterpretationRequest
│   └── status                   p. ej. "complete"
├── source_analysis
│   ├── analysis_engine_version  Copiado de analysis.meta.engine_version
│   ├── analysis_inputs_hash     Copiado de analysis.meta.inputs_hash
│   └── reference                Copiado de analysis.reference
├── primary_problem              Diagnóstico dominante (ver 6.3)
├── priority_corner?             Curva de mayor atención (ver 6.4)
├── patterns[]                   Patrones recurrentes detectados (ver 6.5)
├── insights[]                   1–3 insights priorizados (ver 6.6)
├── assessments[]                Fortalezas y mejoras (ver 6.7)
├── area_evaluations[]           Evaluación por área (ver 6.8)
├── recommendations[]            Plan de entrenamiento (ver 6.9)
├── focus_next                   Una frase: próximo foco
├── overall_grade?               0–10 si las reglas lo permiten; si no, ausente
├── sections[]                   Narrativa compuesta para el informe (ver 6.10)
├── evidence_index[]             Índice machine-readable de evidencia (ver 6.11)
└── warnings[]                   Matices sobre un resultado ya consistente
```

## 6.3 `primary_problem`

| Campo | Notas |
|-------|-------|
| `code` | Etiqueta estable de problema (vocabulario cerrado de reglas) |
| `summary` | Una frase: qué es el problema principal |
| `impact_ms` | Impacto estimado en ms **derivado solo** de hallazgos/métricas de v0.3 |
| `evidence_refs[]` | Claves del `evidence_index` que lo sustentan |

Si no hay evidencia suficiente para afirmar un problema dominante, el Core
**falla** con `InsufficientEvidenceError` o emite interpretación con
`primary_problem.code = "none_detected"` **solo** cuando el análisis muestra
pérdidas/errores nulos de forma coherente (regla explícita; no silencio opaco).

## 6.4 `priority_corner`

| Campo | Notas |
|-------|-------|
| `corner_ref` | Misma clave lógica que en `CornerFinding` |
| `reason` | Por qué merece atención (citando evidencia) |
| `time_loss_ms` | Copiado / agregado desde hallazgos; no recalculado desde telemetría |
| `evidence_refs[]` | Enlaces al índice |

Ausente únicamente si no hay hallazgos atribuibles a curva y la regla lo
documenta (p. ej. solo `unattributed_gains`).

## 6.5 `patterns[]`

Patrones **dentro de la sesión** observables en el `SessionAnalysis`:

| Ejemplo de `code` | Evidencia típica |
|-------------------|------------------|
| `repeated_exit_loss` | Varias curvas con pérdida en el mismo tipo de tramo / categoría |
| `consistency_spread` | `consistency_metrics` por encima de umbral versionado |
| `error_cluster` | Varias etiquetas en `errors_detected` de la misma familia |

Cada patrón lleva `summary`, `evidence_refs[]` y, si aplica, `impact_ms`.

## 6.6 `insights[]` — formato de producto

Máximo **3**. Cada insight sigue `DDP_RULES.md`:

```
INSIGHT     una frase: qué pasó
EVIDENCE    dos o tres datos concretos (vía evidence_refs)
ACTION      qué trabajar en la próxima sesión
```

| Campo | Notas |
|-------|-------|
| `rank` | 1..N; 1 = mayor prioridad |
| `insight` | Frase única |
| `action` | Acción concreta |
| `evidence_refs[]` | ≥ 1; sin evidencia el insight es ilegal |
| `word_budget` | El texto combinado respeta ≤ 150 palabras |

## 6.7 `assessments[]`

Mapeo directo al modelo (`DATA_MODEL.md` §8.3):

| Campo | Valores |
|-------|---------|
| `kind` | `strength` · `improvement` |
| `text` | Juicio breve anclado a evidencia |
| `rank` | Orden de importancia |
| `subject_type` | `corner` · `technique` |
| `corner_ref?` | Obligatorio si `subject_type = corner` |
| `evidence_refs[]` | Trazabilidad |

## 6.8 `area_evaluations[]`

Áreas libres según el modelo (no enum de esquema). En v0.4 el motor emite un
conjunto **inicial cerrado por reglas** (p. ej. Ritmo, Consistencia, Frenada,
Acelerador, Gestión del coche) solo cuando hay métricas/hallazgos que las
soporten. `score_out_of_10` es opcional: si la regla no puede justificar una
nota, el campo queda ausente (hallazgo 6 del modelo).

## 6.9 `recommendations[]` — plan de mejora

| Campo | Notas |
|-------|-------|
| `action` | Qué practicar |
| `target_corner_ref?` | Curva donde practicar |
| `success_criterion` | **Obligatorio** en v0.4: cómo se medirá el éxito con métricas de v0.3 |
| `priority` | Orden de entrenamiento (ver D17) |
| `status` | Siempre `proposed` al emitir (nunca marca cumplimiento por sí solo) |
| `evidence_refs[]` | Justificación (ver D15) |
| `problem_ref?` | Código del problema al que responde (p. ej. `primary_problem.code`) |

Una recomendación sin `success_criterion` o sin evidencia **no puede existir**
en un `SessionInterpretation` (invariante de dominio).

### Invariantes de recomendaciones (D15–D17)

| ID | Invariante |
|----|------------|
| **D15** | Toda recomendación debe ser completamente explicable utilizando **únicamente** la evidencia incluida en el mismo `SessionInterpretation` (`evidence_refs` ⊆ `evidence_index`). No se admite apelar a telemetría, a conocimiento externo ni a evidencia omitida del índice. |
| **D16** | El motor **nunca** emite recomendaciones mutuamente contradictorias sobre un mismo problema. Dos recomendaciones con el mismo `problem_ref` no pueden prescribir acciones opuestas ni criterios de éxito incompatibles. Ante conflicto de reglas, gana la de mayor `priority_weight` documentado; la otra no se emite. |
| **D17** | La prioridad de las recomendaciones es completamente determinista y depende **únicamente** de reglas documentadas (`priority_weight`, empates por `rule_id` / `action` lexicográficos), nunca del orden interno de procesamiento ni de estructuras de hash/set no ordenadas. El campo `priority` resultante es denso: `1..N` sin huecos. |

Estas recomendaciones **no** crean filas en `objective`. La aceptación de un
objetivo permanente es un acto de usuario fuera del Core (alcance 2.2).

## 6.10 `sections[]`

Narrativa abierta del informe (`key`, `title`, `body`, `order`), lista para
persistir. Claves mínimas de v0.4:

| `key` | Contenido |
|-------|-----------|
| `executive_summary` | Resumen de la sesión en pocas frases |
| `engineer_analysis` | Análisis del problema principal y patrón |
| `highlight` | Mensaje corto de foco |
| `traceability` | Cuerpo canónico (JSON UTF-8) del `evidence_index` + metadatos fuente |

`traceability` garantiza que el documento persistido conserve la cadena de
prueba sin añadir tablas ni columnas (secciones abiertas del modelo).

## 6.11 `evidence_index[]` — trazabilidad machine-readable

Cada entrada:

```
EvidenceItem
├── ref                  Clave estable dentro del informe (p. ej. "E1")
├── kind                 corner_finding | unattributed_gain | metric |
│                        error_label | reference | analysis_meta
├── path                 Ruta lógica dentro del SessionAnalysis
│                        (p. ej. "corner_findings[Forrest's Elbow].time_loss_ms")
├── value                Valor citado (escalar / etiqueta)
├── analysis_engine_version
└── analysis_inputs_hash
```

Reglas:

1. Toda conclusión (`primary_problem`, insight, assessment, recommendation,
   pattern, `focus_next`) referencia ≥ 1 `EvidenceItem`.
2. Todo `EvidenceItem` apunta a un dato presente en el `InterpretationRequest.analysis`.
3. Si una regla no puede citar evidencia, **no emite** la conclusión; no inventa.
4. El índice completo se copia a `sections[key=traceability]`.

## 6.12 Qué queda fuera del contrato (a propósito)

| Dato | Quién lo aporta |
|------|-----------------|
| `session_id` · `driver_id` · `report_id` enteros | Backend |
| `report_number` | Backend (política de numeración, apartado 9.3) |
| `corner_id` entero | Backend al resolver `corner_ref` |
| `driver_index_id` | Fuera de v0.4 |
| Bytes / canales de telemetría | Prohibidos |
| Diagnóstico de fallo | Error estructurado, no un campo del éxito |

## 6.13 Errores de dominio (fallo)

| Error | Cuándo |
|-------|--------|
| `MissingAnalysisError` | El request no trae un análisis completo usable |
| `IncompatibleAnalysisError` | Falta `engine_version` / `inputs_hash` / referencia / status |
| `InsufficientEvidenceError` | No hay base objetiva para emitir juicio útil |
| `InvalidInterpretationOptionsError` | Opciones fuera de rango (p. ej. `max_insights > 3`) |
| `InconsistentInterpretationError` | Resultado estructuralmente incoherente (rompe invariantes) |

Un warning **nunca** sustituye a un fallo.

## 6.14 Determinismo del motor (D14-I)

El motor es **determinista**:

```
mismas entradas (InterpretationRequest canónico)
  + misma engine_version de interpretación
  ⇒ mismo SessionInterpretation
```

Mismos códigos, mismos ranks, mismos textos emitidos por plantilla, mismo
`inputs_hash`. La única excepción admitida es `computed_at`.

---

# 7. Estrategia de interpretación

## 7.1 Cadena interna (solo juicio sobre medición)

```
SessionAnalysis (request.analysis)
      │
      ▼
 evidence binding              indexa hallazgos, métricas, errores, referencia
      │
      ▼
 coach.py                      problema principal · curva · patrones · insights
      │
      ▼
 recommendations.py            plan ordenado + success_criterion
      │
      ▼
 reports/session.py            sections · focus_next · overall_grade?
      │
      ▼
 SessionInterpretation         contrato público
```

Ninguna etapa:

- lee telemetría,
- llama a `analyze(...)`,
- salta hacia `telemetry/analyzer.py` para recalcular,
- consulta SQLite,
- invoca un modelo generativo opaco.

## 7.2 Reglas de interpretación (deterministas)

v0.4 implementa un **catálogo versionado de reglas** (`interpretation_rules`),
no un modelo opaco.

Cada regla declara:

| Campo | Propósito |
|-------|-----------|
| `rule_id` | Identidad estable |
| `applies_when` | Predicado sobre campos del `SessionAnalysis` |
| `emits` | Qué conclusiones puede producir |
| `priority_weight` | Peso para ordenar insights / foco |
| `evidence_paths` | Qué rutas debe citar |

Ejemplos de política inicial (cerrados en implementación, no aquí inventados
como números mágicos sin prueba):

1. **Curva prioritaria** = hallazgo con mayor `time_loss_ms` positivo; empates
   por `estimated_gain_ms` y luego por `corner_ref` lexicográfico.
2. **Problema principal** = regla de mayor peso cuyo predicado se cumple
   (p. ej. pérdida dominante en una curva vs. dispersión de consistencia vs.
   cluster de errores).
3. **Fortalezas** = curvas con `time_loss_ms` negativo relevante y/o ausencia
   de errores cuando el análisis lo documenta de forma positiva.
4. **Recomendaciones** = una por insight de rango 1..N, con
   `success_criterion` expresado en métricas de v0.3
   (p. ej. “reducir `time_loss_ms` en Forrest's Elbow respecto a la misma
   referencia en la próxima sesión”).

Los umbrales numéricos concretos se fijan en el código versionado del motor y
quedan cubiertos por el `engine_version` + tests de fase. Cambiar umbrales =
subir `engine_version`.

## 7.3 Qué significa “interpretar, no medir” en la práctica

| Permitido en `SessionInterpretation` | Prohibido (sigue en v0.3) |
|--------------------------------------|---------------------------|
| "Forrest's Elbow es la curva prioritaria porque `time_loss_ms = 180`" | Recalcular `time_loss_ms` desde canales |
| "Consistencia a mejorar porque `stdev_ms = 240`" | Releer vueltas del `.ibt` |
| "Trabajar salida de curva; éxito = pérdida < X ms" | Inventar una pérdida no presente en findings |
| `errors_detected` → insight de bloqueo | Detectar un lockup nuevo ausente en el análisis |

## 7.4 Composición del informe

`reports/session.py` no decide prioridades: **compone** a partir de la salida
de `coach.py` y `recommendations.py`. Si la composición necesitara un juicio
nuevo, el diseño está mal: ese juicio pertenece a `engineer/`.

## 7.5 Relación con objetivos

- v0.4 puede **mencionar** en narrativa que una recomendación es candidata a
  objetivo.
- v0.4 **no** escribe `objective` ni `objective_progress`.
- Cuando exista aceptación de usuario, el Backend creará `Objective` con
  `source_analysis_report_id` apuntando al informe ya persistido — fuera del
  Core y, si se desea en esta etapa, como fase API opcional posterior a la
  congelación (decisión abierta A3).

---

# 8. Versionado del motor

## 8.1 Tres ejes independientes

| Eje | Qué cambia | Cómo se resuelve |
|-----|-----------|------------------|
| Versión de esquema SQLite | Forma de las tablas | Migración (cerrada en v0.1) |
| `Analysis.engine_version` | Fórmula de medición (v0.3) | Recálculo selectivo de `analysis` |
| `AnalysisReport.engine_version` | Reglas de interpretación (v0.4) | Regeneración en el sitio del informe |

No se confunden. Mejorar el juicio **no** es una migración ni un reanálisis.

## 8.2 Formato de `engine_version` (interpretación)

Cadena estable y comparable, p. ej. `ddp-interpretation-0.4.0`.

Reglas:

1. La emite el Core de interpretación (constante de entrega).
2. El Backend la persiste en `analysis_report.engine_version` tal cual.
3. Misma versión + mismos inputs ⇒ mismo `inputs_hash` y mismo resultado
   interpretable.
4. Al subir la versión, el informe existente se **actualiza en el sitio** al
   regenerar; no se crea un segundo `analysis_report` con otro `id` para la
   misma identidad natural.

## 8.3 `inputs_hash` (interpretación)

Huella de las **entradas del juicio**, no de la telemetría cruda.

Incluye, de forma canónica (orden estable, UTF-8, sin rutas ni IDs enteros):

- `analysis.meta.engine_version` y `analysis.meta.inputs_hash`,
- proyección canónica de findings, métricas, errores, referencia y attributes
  del análisis usados,
- `session_context` canónico,
- opciones (`max_insights`, …),
- `engine_version` de interpretación.

**Decisión (D6):** el Core calcula `inputs_hash` sobre el
`InterpretationRequest` canónico y lo devuelve en
`SessionInterpretation.meta`. El Backend lo persiste.

Consecuencia: si cambia el análisis de v0.3 (otro `inputs_hash` o
`engine_version`), el hash de interpretación cambia y el informe queda
obsoleto de forma detectable.

## 8.4 Relación con el Roadmap §3

`algorithm_version` y `parameters_hash` quedan **fuera de v0.4**. Con un solo
catálogo de reglas y sin parámetros configurables por usuario,
`engine_version` + `inputs_hash` bastan.

---

# 9. Política de regeneración de derivados

Heredada de `DATABASE_DESIGN.md` §2.4 y §7; aquí solo se operacionaliza para
v0.4.

## 9.1 Grado de `analysis_report`

`analysis_report` es **regenerable en el sitio**:

- se recalcula el contenido,
- se conservan `id` y `report_number`,
- las hijas se sustituyen enteras,
- nunca se borra el padre para volver a crearlo.

Razones ya congeladas: (1) destino futuro/actual de
`objective.source_analysis_report_id`; (2) documento que el piloto leyó.

## 9.2 Cascada lógica desde `analysis`

```
analysis (desechable)
    → analysis_report (regenerable en el sitio)
```

| Evento | Comportamiento en v0.4 |
|--------|------------------------|
| Interpretación pedida sin informe previo | Crear informe y enlazar sesión |
| Reinterpretar con mismo `inputs_hash` y `force=false` | `outcome = unchanged` |
| Reinterpretar con inputs distintos o `force=true` | Actualizar en el sitio |
| `Analysis` regenerado (nuevo hash/versión) | El informe queda obsoleto; regenerar interpretación actualiza el mismo `id` |
| `Analysis` ausente / no completo | No interpretar; error de orquestación |
| Borrado de sesión | Cascada ya definida en v0.1 sobre el puente; no hay acción extra de juicio |

v0.4 **no** regenera `analysis`. Si el análisis falta, se indica al usuario que
ejecute v0.3.

## 9.3 Numeración de `report_number` (Backend)

Clave natural congelada: `(scope, report_number)`.

Política de v0.4 para `scope = session`:

1. Si ya existe un informe de sesión enlazado a esa `session_id` vía
   `analysis_report_session`, se reutiliza su `report_number` (regeneración en
   el sitio).
2. Si no existe, el Backend asigna el siguiente `report_number` disponible para
   `scope = session` (monótono, positivo).
3. El Core **no** conoce `report_number`.

## 9.4 Qué nunca se regenera desde el motor de interpretación

- `Session`, `Lap`, `TelemetryFile`, métricas de sesión
- `Analysis` / `analysis_corner_finding`
- Catálogos
- `Objective` (definición)
- `DriverIndex`, `TrackProfile`, `Career` (fuera de alcance)

## 9.5 Atomicidad

Persistir un informe ocurre dentro de la transacción del repositorio (`store`
actualiza padre y sustituye hijas). Un fallo deja la versión anterior intacta;
nunca un informe a medias.

---

# 10. Organización interna de módulos

Nada de esto se implementa hasta congelar este documento. Es el mapa de destino.
Los stubs actuales se convierten en módulos reales **solo** en la medida del
alcance de v0.4.

```
core/engineer/
├── __init__.py
├── models.py                 InterpretationRequest · SessionInterpretation · EvidenceItem
├── errors.py                 Errores de dominio de interpretación
├── hashing.py                Canonicalización + inputs_hash
├── evidence.py               Construcción del evidence_index
├── rules.py                  Catálogo versionado de reglas deterministas
├── coach.py                  Problema · curva · patrones · insights · assessments
└── recommendations.py        Plan de entrenamiento + success_criterion

core/reports/
├── session.py                Composición sections / focus_next / grade
├── weekly.py                 (stub; fuera de v0.4)
└── career.py                 (stub; fuera de v0.4)

backend/interpretation/       (paquete de orquestación)
├── __init__.py
├── pipeline.py               load Analysis → request → Core → resolve → store
├── project.py                Analysis row + findings → InterpretationRequest
└── service.py                Fachada para endpoints

backend/api/
└── interpretation_routes.py  Endpoints HTTP de interpretación / informe de sesión
```

### Qué permanece stub o cerrado

```
core/telemetry/*              v0.2 / v0.3 — cerrados; solo se leen resultados
core/scoring/*                posterior (DriverIndex)
core/objectives/*             posterior (aceptación de objetivos)
core/reports/weekly.py        posterior
core/reports/career.py        posterior
```

La carpeta `backend/db/` **no se reestructura**. El orquestador la consume
(`AnalysisRepository`, `AnalysisReportRepository`, `SessionRepository`,
`CatalogRepository`).

### Frontera interna del Core

```
telemetry/   mide
engineer/    interpreta y prioriza
reports/     compone
```

Sin mezclar. `reports/session.py` no prioriza. `engineer/*` no maqueta más allá
de los campos del contrato. Ninguno mide.

---

# 11. Superficie HTTP y UI

## 11.1 API mínima

| Método | Ruta | Función |
|--------|------|---------|
| `POST` | `/sessions/{id}/interpretation` | Interpretar o reinterpretar la sesión |
| `GET` | `/sessions/{id}/interpretation` | Leer interpretación / informe de sesión |
| `GET` | `/sessions/{id}/interpretation/insights` | Insights priorizados |
| `GET` | `/sessions/{id}/interpretation/recommendations` | Plan de entrenamiento |
| `POST` | `/interpretation/regenerate` | Lote: regenerar informes obsoletos / faltantes |
| `GET` | `/interpretation/engine` | `engine_version` vigente del motor de interpretación |

Cuerpo lógico de `POST /sessions/{id}/interpretation`:

```
{
  analysis_engine_version?: string,  // por defecto: análisis más reciente completo
  force?: boolean                    // regenerar aunque inputs_hash coincida
}
```

Respuesta lógica:

```
{
  session_id,
  report_id,
  report_number,
  scope: "session",
  engine_version,                 // interpretación
  inputs_hash,
  status,
  source_analysis: {
    engine_version,
    inputs_hash,
    reference
  },
  primary_problem,
  priority_corner,
  insights,
  assessments,
  recommendations,
  focus_next,
  overall_grade,
  warnings,
  outcome: "created" | "updated" | "unchanged" | "failed"
}
```

`outcome = unchanged` cuando `force` es falso y ya existe informe de la sesión
con el mismo `inputs_hash` de interpretación.

`outcome = updated` (no `replaced`) refleja la política de regeneración en el
sitio: la identidad del informe no cambia.

## 11.2 Frontend mínimo

En la vista de sesión (junto al panel de análisis de v0.3):

1. Botón “Interpretar” / “Reinterpretar” (habilitado solo si hay análisis).
2. Estado: sin interpretación · calculando · completo · sin análisis previo.
3. Bloque de insights (máx. 3) en formato Insight / Evidencia / Acción.
4. Curva prioritaria y problema principal.
5. Lista de fortalezas / mejoras.
6. Plan de recomendaciones con criterio de éxito.
7. Enlace o panel al informe de sesión compuesto.

Sin Driver Index, sin informe semanal, sin edición de objetivos permanentes.

---

# 12. Decisiones de diseño de v0.4

Decisiones que este documento **toma** y que, tras la congelación, quedan fijas
para la implementación.

| ID | Decisión | Elección |
|----|----------|----------|
| **D1** | Alcance de v0.4 | Solo interpretación de sesión a partir de `SessionAnalysis`. Sin medición |
| **D2** | Contrato Core → Backend | `SessionInterpretation` es el único objeto de éxito; fallo = error estructurado |
| **D3** | Paquete de orquestación | `backend/interpretation/` |
| **D4** | Frontera de IDs | El Core usa `corner_ref` / `session_key`; el Backend resuelve enteros y `report_number` |
| **D5** | Persistencia | Solo `AnalysisReportRepository` v0.1; política **actualizar en el sitio** |
| **D6** | `inputs_hash` | Lo calcula el Core sobre el request canónico; incluye hash/versión del análisis fuente |
| **D7** | Motor determinista | Catálogo de reglas explícitas; sin LLM / juicio opaco |
| **D8** | Techo de insights | Máximo 3 (`DDP_RULES.md`) |
| **D9** | Evidencia obligatoria | Ninguna conclusión sin `evidence_refs` → `evidence_index` |
| **D10** | Recomendaciones | Siempre con `success_criterion` medible en términos de v0.3; status inicial `proposed` |
| **D11** | Objetivos permanentes | No se escriben en v0.4; solo se proponen como recomendaciones |
| **D12** | Atomicidad del resultado del Core | Éxito completo o error; nunca parcial |
| **D13** | Trazabilidad cruzada | Toda evidencia cita `analysis_engine_version` + `analysis_inputs_hash` |
| **D14** | Determinismo | Mismas entradas + misma `engine_version` ⇒ mismo `SessionInterpretation` (igualdad salvo `computed_at`) |
| **D15** | Explicabilidad de recomendaciones | Toda recomendación se explica solo con la evidencia del propio `SessionInterpretation` (apartado 6.9) |
| **D16** | No contradicción | Nunca se emiten recomendaciones mutuamente contradictorias sobre un mismo problema (apartado 6.9) |
| **D17** | Prioridad determinista | El orden de recomendaciones depende solo de reglas documentadas, nunca del orden de procesamiento (apartado 6.9) |
| **D18** | `DriverIndex` / weekly / career / TrackProfile | Fuera de v0.4 |
| **D19** | Disparo | Explícito por API/UI; no obligatorio post-análisis en el cierre mínimo |
| **D20** | Sección `traceability` | Persiste el índice de evidencia en `sections[]` sin cambio de esquema |

---

# 13. Decisiones abiertas (no bloquean el diseño)

| ID | Tema | Notas | Cuándo decidir |
|----|------|-------|----------------|
| A1 | Umbrales numéricos iniciales de reglas | Valores concretos de peso / relevancia | Fase 2–3 + fixtures |
| A2 | Vocabulario cerrado de `primary_problem.code` y `pattern.code` | Lista inicial vs extensible | Fase 1 |
| A3 | Endpoint de aceptación de recomendación → `Objective` | Producto útil, no necesario para cerrar el motor | Post-v0.4 o fase API extra |
| A4 | Interpretación automática tras análisis v0.3 | Conveniencia; no cambia el motor | Post-cierre o fase UI |
| A5 | `overall_grade` | Emitir nota 0–10 con regla explícita vs dejar ausente en v0.4 | Fase 3 |
| A6 | Idioma de textos emitidos | Castellano de las fuentes vs inglés de código | Fase 1 (textos al piloto) |

Las decisiones pendientes 1–3 de `DATA_MODEL.md` (fórmula del índice,
dimensiones, error score) **no se reabren** aquí: pertenecen a scoring y quedan
fuera de v0.4 (D18).

---

# 14. Fases de implementación

Misma metodología que v0.1–v0.3: una fase → implementación → verificación →
revisión → aprobación. **No se avanza de fase sin aprobación.**

Cada fase se valida con un criterio falsable. Si no lo cumple, no está
terminada.

| # | Fase | Entrega | Verificación |
|:-:|------|---------|--------------|
| **0** | Contrato `SessionInterpretation` | `engineer/models.py`, errores, superficie `interpret(request) -> SessionInterpretation`; invariante D12 | Estructuras importables; Core sin SQLite/HTTP/rutas/repositorios/IDs enteros/bytes; no existe campo de errores dentro del éxito; request sin telemetría |
| **1** | Evidencia y reglas base | `evidence.py` + `rules.py` (catálogo mínimo) | Todo ítem de evidencia apunta a paths existentes del análisis fixture; regla sin match no emite; vocabulario A2 inicial cerrado |
| **2** | Coach | `coach.py`: primary_problem, priority_corner, patterns, insights, assessments | ≤ 3 insights; cada uno con evidence_refs; curva prioritaria = mayor pérdida del fixture; sin texto sin ancla |
| **3** | Recomendaciones y composición | `recommendations.py` + `reports/session.py` | Toda recomendación trae success_criterion; cumple D15–D17; sections mínimas presentes; `traceability` redondeable a evidence_index; sin llamadas a telemetry/ |
| **4** | Hash y versión | `hashing.py` + `engine_version` estable | Mismo request ⇒ mismo `inputs_hash`; cambiar un finding cambia el hash; versión no vacía; igualdad salvo `computed_at` |
| **5** | Orquestador | `backend/interpretation/pipeline.py` + proyección + `AnalysisReportRepository.store` | Sesión con Analysis → 1 report + hijas; reinterpretar actualiza mismo `id`/`report_number`; sin Analysis → no persiste; no escribe `analysis` |
| **6** | API HTTP | Rutas de interpretación / lectura / regeneración | `POST` crea o actualiza; `GET` coincide con BD; `unchanged` si hash idéntico; regresión análisis v0.3 verde |
| **7** | UI mínima | Disparo + insights + plan + estado | Desde la app: con análisis fixture → ver problema/curva/recomendaciones; sin análisis → CTA a analizar |
| **8** | Verificación de etapa | Script `verify_v0_4.py` + regresión v0.1/v0.2/v0.3 | E2E verde; documentos congelados intactos; `CHANGELOG.md` listo para cerrar la etapa |

### Por qué este orden

- **0 → 4** construyen el Core de dentro afuera, testeable sin FastAPI ni UI.
- **5** añade el adaptador de persistencia respetando regeneración en el sitio.
- **6 → 7** exponen el valor al usuario solo cuando el juicio ya es trazable.
- **8** cierra la etapa con la misma disciplina de regresión que cerró v0.3.

### Fixtures

v0.4 reutiliza el análisis producido por el fixture de v0.3 y añade:

1. Oráculo de interpretación esperada (problema principal, curva prioritaria,
   ranks, códigos de regla).
2. Casos negativos: sin análisis; análisis incompleto; findings vacíos con
   métricas coherentes / incoherentes.

Sin oráculo de interpretación la fase 2 no es aprobable.

---

# 15. Criterios de cierre de v0.4

v0.4 se considera **completa** cuando:

1. Este documento está aprobado y congelado.
2. Las fases 0–8 están implementadas, verificadas y aprobadas una a una.
3. Una sesión con `Analysis` completo se interpreta de extremo a extremo desde
   la UI o la API.
4. El resultado persiste en `analysis_report` + hijas con trazabilidad completa
   e índice de evidencia.
5. Reinterpretar actualiza el mismo informe; no crea un `id` nuevo para la
   misma sesión.
6. Interpretar sin análisis no corrompe hechos ni inventa mediciones.
7. La salida del Core no contiene canales de telemetría ni recalcula métricas
   de v0.3.
8. Toda conclusión del oráculo de fixture cita evidencia verificable.
9. La capa `backend/db` no ha cambiado, salvo fixes demostrables documentados.
10. Los documentos congelados de v0.1–v0.3 permanecen idénticos.
11. La regresión v0.1, v0.2 y v0.3 sigue verde.
12. `CHANGELOG.md` registra v0.4 como etapa cerrada.

---

# 16. Relación con el roadmap y etapas vecinas

| Documento / punto | Efecto de v0.4 |
|-------------------|----------------|
| `ANALYSIS_ENGINE.md` | Queda cerrado; v0.4 solo consume su salida |
| `IMPORT_PIPELINE.md` | Queda cerrado; sin relectura de ficheros |
| Roadmap §3 Trazabilidad fina | Sigue aplazado; v0.4 usa `engine_version` + `inputs_hash` |
| Decisiones pendientes 1–3 del modelo | Siguen abiertas; DriverIndex fuera de alcance |
| README · Ingeniero IA | Esta etapa materializa el núcleo de sesión de ese hito |

---

# 17. Necesidades que tocarían documentos congelados

Durante este diseño **no** se ha encontrado ninguna necesidad que obligue a
modificar un documento congelado.

Puntos vigilados y resolución **sin** abrir v1.1:

| Tensión | Resolución dentro de lo congelado |
|---------|-----------------------------------|
| Evidencia estructurada sin tabla propia | `evidence_index` en el contrato Core + sección `traceability` (`sections[]` abiertas) |
| `DriverIndex` en el grafo `analysis → driver_index → analysis_report` | `driver_index_id` opcional en esquema; v0.4 deja `NULL` (D18) |
| Objetivos propuestos vs permanentes | Solo `recommendations[]` con `status=proposed`; no se escribe `objective` |
| Cascada analysis → report | Detectable por inclusión del hash/versión de análisis en `inputs_hash`; regeneración explícita |
| LLM / “Ingeniero IA” como modelo generativo | Interpretación determinista por reglas (D7); cumple “no inventar” y trazabilidad |
| Informe semanal / career / TrackProfile | Fuera de alcance; stubs intactos |

Si en revisión o implementación aparece una necesidad real de cambiar un
congelado, **se detiene el trabajo**, se justifica y se espera aprobación
explícita antes de continuar.

---

# 18. Reglas de uso de este documento

1. La implementación de v0.4 se deriva de este documento, y este documento se
   deriva de los documentos congelados. La cadena no se recorre al revés.
2. Una fase no puede introducir una responsabilidad que contradiga los
   apartados 4–6 (frontera y contrato `SessionInterpretation`).
3. Si hace falta un atributo de dominio nuevo, se detiene el trabajo y se abre
   v1.1 del modelo o del diseño de BD.
4. **Ninguna modificación estructural del diseño durante la implementación.**
   Ante una necesidad nueva, el desarrollo se detiene y se revisa este
   documento con re-aprobación explícita.
5. Ninguna fase se da por terminada sin cumplir su verificación.
6. Este documento no tiene autoridad para modificar documentos congelados.
7. La implementación se deriva de este documento; no al revés.

---

# 19. Cierre del diseño

## Estado del documento

| Aspecto | Situación |
|---------|-----------|
| Base | Documentos congelados de v0.1–v0.3, sin modificar |
| Contrato Core → Backend | `SessionInterpretation` |
| Entrada | `InterpretationRequest` (sin telemetría) |
| Invariante de éxito | D12: objeto completo o error estructurado; nunca parcial |
| Trazabilidad | D9 + D13 + D15 + sección `traceability` (D20) |
| Determinismo | D14 + D17 |
| No contradicción | D16 |
| Frontera de capas | Explícita en el apartado 4 |
| Decisiones cerradas | D1–D20 |
| Decisiones abiertas | A1–A6, ninguna bloqueante |
| Implementación | En curso — se deriva de este documento |

**Versión 1.0 · congelada · referencia oficial de la etapa v0.4.**

---

*Documento congelado. La implementación se deriva de aquí; no al revés.*
