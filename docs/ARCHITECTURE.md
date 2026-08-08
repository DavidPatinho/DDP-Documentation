# DDP — Arquitectura

> Documento de arquitectura del sistema.
>
> **Norte de producto:** `DDP_PHILOSOPHY.md` — qué es DDP y qué merece existir.
> Complementa `DDP_RULES.md` (diseño, convenciones y disciplina de ingeniería)
> y `COACH_PHILOSOPHY.md` (cómo piensa el Coach).
>
> **Memoria del proyecto:** `docs/project/` — PMS + AICS.
> Forman parte oficial de la arquitectura de desarrollo. No son opcionales.
>
> Ante conflicto de *misión o criterio de producto*, manda `DDP_PHILOSOPHY.md`.
> Ante conflicto de *reglas técnicas o de UI*, manda `DDP_RULES.md`.
> Ante conflicto de *estado actual / relevo*, mandan `docs/project/PROJECT_STATE.md`
> y `docs/project/HANDOVER.md`.

---

## Capas del sistema (visión completa)

```text
Development Infrastructure          tools/ · scripts de desarrollo
        ↓
Project Memory System (PMS)         docs/project/ — memoria oficial
        ↓
AI Context System (AICS)            BOOTSTRAP / AI_BOOTSTRAP / config
        ↓
Product Architecture                Tauri → Frontend → Backend → Core → SQLite
```

El AICS pertenece a la **infraestructura oficial de desarrollo**.
No forma parte del runtime del piloto.

---

## Visión general (Product Architecture)

DDP se organiza en capas con responsabilidades estrictamente separadas.
El elemento central es el **Core**: un motor de dominio en Python que no
depende del frontend ni del backend.

```
┌──────────────────────────────────────────────────────────┐
│  TAURI — shell nativo de escritorio (Windows)            │
│  Ventana, ciclo de vida, acceso a sistema de archivos    │
└──────────────────────────────────────────────────────────┘
                            │
┌──────────────────────────────────────────────────────────┐
│  FRONTEND — React + TypeScript + Tailwind + shadcn/ui    │
│  Presentación, navegación, interacción, visualización    │
│  NO contiene lógica de dominio · NO interpreta causalidad│
└──────────────────────────────────────────────────────────┘
                            │  HTTP / JSON
┌──────────────────────────────────────────────────────────┐
│  BACKEND — Python FastAPI                                │
│  Endpoints, validación, orquestación, persistencia       │
│  NO contiene lógica de dominio: la delega al Core        │
└──────────────────────────────────────────────────────────┘
                            │  llamadas Python directas
┌──────────────────────────────────────────────────────────┐
│  CORE — motores deterministas (Pandas / NumPy)           │
│  Telemetría, análisis, interpretación, attribution,      │
│  strategy, training · Independiente de HTTP, UI y BD     │
└──────────────────────────────────────────────────────────┘
                            │
┌──────────────────────────────────────────────────────────┐
│  SQLITE — persistencia local                             │
└──────────────────────────────────────────────────────────┘
```

Pipeline oficial de producto (DDP 4.0):

```text
Import → Analysis → Interpretation
       → Attribution → Strategy Facts → Development Strategy
       → Planner → Coach → Dashboard → Journey
```

Una pregunta por motor (ver `COACH_PHILOSOPHY.md`):

| Motor | Pregunta |
|-------|----------|
| Analysis | ¿Qué ocurrió? |
| Interpretation | ¿Qué problema priorizar en esta sesión? |
| Attribution | ¿Por qué ocurrió? |
| Development Strategy | ¿Qué debo hacer ahora? |
| Training Planner | ¿Cómo entreno hoy? |
| Progress / Profile | ¿Ha funcionado? ¿Cómo evoluciono? |
| Coach / Dashboard / Journey | ¿Cómo se lo digo al piloto sin contradecir los hechos? |

---

## Project Memory System (PMS) + AICS + PMI

El PMS es la **memoria oficial** de DDP. Vive en `docs/project/`.

La **PMI** (`tools/pms/`) valida y orquesta el mantenimiento del PMS.

El **AICS** (AI Context System) es la puerta de entrada:

| Audiencia | Entrada |
|-----------|---------|
| Humanos | `docs/project/BOOTSTRAP.md` |
| IA | `docs/project/AI_BOOTSTRAP.md` |

El AICS no sustituye al PMS: resume y enlaza para reconstrucción rápida.
Especificación: `docs/project/PMS_SPEC.md`. Herramientas: `tools/README.md`.

| Documento | Función |
|-----------|---------|
| `BOOTSTRAP.md` | Entrada humana (AICS) |
| `AI_BOOTSTRAP.md` | Entrada IA + protocolo + ChatGPT Startup Prompt |
| `PROJECT_CONFIGURATION.md` | Configuración global (única fuente de URLs/repos) |
| `PROJECT_CONTEXT.md` | Incorporación |
| `PROJECT_STATE.md` | Estado actual |
| `DECISIONS.md` | Decisiones de producto |
| `ADR.md` | Decisiones técnicas |
| `ROADMAP.md` | Evolución de producto |
| `KNOWN_ISSUES.md` | Incidencias abiertas |
| `HANDOVER.md` | Relievo (única versión viva) |
| `SESSION_NOTES.md` | Diario cronológico (append-only) |
| `CHANGELOG_AI.md` | Cambios funcionales |
| `AI_GUIDE.md` | Papel de la IA |
| `PMS_SPEC.md` | Especificación oficial PMS/PMI/AICS |
| `PHASE_CLOSE_CHECKLIST.md` | Checklist reutilizable de cierre |

### Procedimiento oficial: `/close-phase`

```text
Desarrollar → Tests → validate_pms → actualizar PMS
→ HANDOVER → Issues → STATE → CHANGELOG → ADR/DECISIONS
→ AICS (AI_BOOTSTRAP + ChatGPT prompt desde PROJECT_CONFIGURATION)
→ sync_docs → validación final → CERRAR
```

```bash
py -m tools.pms.close_phase --phase "Nombre de la fase"
py -m tools.pms.validate_pms
```

Ninguna fase se considera terminada hasta que el procedimiento complete.
Código verde + tests verdes **no bastan**.

### Orden de arranque

- **IA:** `AI_BOOTSTRAP.md` → profundizar solo con enlaces del PMS.
- **Humano:** `BOOTSTRAP.md` → luego lectura profunda:

1. `DDP_PHILOSOPHY.md`
2. `ARCHITECTURE.md` (este)
3. `docs/project/PROJECT_CONTEXT.md`
4. `docs/project/PROJECT_STATE.md`
5. `docs/project/DECISIONS.md`
6. `docs/project/ADR.md`
7. `docs/project/HANDOVER.md`
8. `docs/project/KNOWN_ISSUES.md`

### Papel de la IA

La IA es la **voz** de DDP, nunca su **cerebro**.
Detalle normativo: `docs/project/AI_GUIDE.md` y ADR-002.

```text
Motores deterministas → AI Communication Layer → Conversation
                     → Real Time Engineer → Mentor Deportivo
```

### Futuro: documentación pública

```text
Repo privado (código + docs/) → sync_docs / publish_docs → repo público (solo docs)
```

Preparado en PMI; publicación remota **no** implementada en v1.

---

## El Core como motor independiente

El Core es el cerebro de DDP y la pieza de mayor valor a largo plazo. Contiene
todo el conocimiento de dominio del producto: cómo se lee la telemetría, cómo
se mide el rendimiento, cómo se detecta una limitación y cómo se convierte en
una recomendación útil.

### Independencia estricta

El Core **no conoce ni importa** ninguna de estas tecnologías:

| No depende de | Motivo |
|---------------|--------|
| FastAPI, HTTP, requests | El transporte es una decisión externa al dominio |
| React, DOM, Tauri | La presentación es intercambiable |
| SQLite, ORMs, sesiones de BD | La persistencia es responsabilidad del backend |

Solo depende de **Pandas**, **NumPy** y la biblioteca estándar de Python.

### Dirección de dependencias

Las dependencias apuntan siempre hacia dentro. El Core es el centro y nunca
mira hacia fuera.

```
frontend  ──>  backend  ──>  core
```

El backend importa el Core. El Core nunca importa el backend.

### Qué habilita esta independencia

- **Testeable en aislamiento** — se ejecuta sin levantar servidor ni interfaz
- **Reutilizable** — accesible desde scripts, notebooks o herramientas CLI
- **Sustituible en los extremos** — cambiar de framework web o de interfaz no
  obliga a reescribir lógica de dominio
- **Longevidad** — la lógica de análisis sobrevive a los ciclos de vida de los
  frameworks que la rodean
- **Razonamiento aislado** — un cambio en el análisis de neumáticos no puede
  romper una ruta HTTP

### Contrato del Core

| Aspecto | Definición |
|---------|------------|
| Entrada | Datos y parámetros como estructuras Python o DataFrames |
| Salida | Estructuras de datos puras (dicts, dataclasses, DataFrames) |
| Efectos | Ninguno: no escribe en BD, no hace red, no toca la UI |
| Errores | Excepciones de dominio propias, nunca códigos HTTP |

El Core nunca devuelve HTML, JSON serializado ni componentes. Devolver datos
puros permite que el mismo resultado se renderice en pantalla, se exporte a
PDF o se consuma desde un script sin duplicar lógica.

---

## Estructura del Core

```
core/
├── telemetry/              Lectura y análisis de datos de pista
│   ├── parser.py / iracing/  Ingesta .ibt y normalización
│   ├── analyzer.py         Medición de sesión
│   └── …                   Consistencia, series, corners, etc.
│
├── engineer/               Interpretation Engine (sesión)
│   ├── coach.py            Diagnóstico e insights
│   └── recommendations.py  Acciones con success_criterion
│
├── attribution/            Performance Attribution (multi-sesión)
│   ├── engine.py           ¿Por qué cambió el rendimiento?
│   └── models.py           PerformanceAttribution
│
├── development_strategy/   Development Strategy Engine
│   └── engine.py           ¿Qué debo hacer ahora? (on-demand)
│
├── training/               Skills, planner, progress, perfil, circuit map
│
├── reports/                Composición de informes (session; weekly/career stubs)
├── scoring/                Driver score (evolución futura)
├── objectives/             Objetivos
└── utils/                  Utilidades transversales
```

### Flujo de procesamiento

```
   fichero de telemetría
            │
            ▼
   telemetry/                 normaliza y mide (Analysis)
            │
            ▼
   engineer/                  diagnostica sesión (Interpretation)
            │
            ▼
   training/*                 skills / plan / progress / profile
            │
            ▼
   attribution/               explica causas (PerformanceAttribution)
            │
            ▼
   development_strategy/      decide foco (DevelopmentStrategy)
            │
            ▼
   backend  ──>  frontend     serializa y proyecta (Coach / Dashboard / Journey)
```

Dashboard y Coach **no** inventan causalidad ni foco: proyectan
`PerformanceAttribution` y `DevelopmentStrategy`.

### Reglas internas del Core

1. Un módulo tiene una responsabilidad y la documenta en su docstring
2. `telemetry/` mide, `engineer/` diagnostica sesión, `attribution/` explica,
   `development_strategy/` decide foco, `training/` planifica entrenamiento —
   sin mezclar
3. `utils/` solo admite lo compartido por dos o más módulos
4. Ningún módulo del Core accede a base de datos ni a red
5. Los límites de tamaño de `DDP_RULES.md` aplican igual que en el frontend
6. Planner **nunca** cambia Strategy (ADR-008)
7. Cronología por `session.started_at`, nunca por orden de importación (ADR-005)

---

## Responsabilidades por capa

### Tauri

Contenedor nativo. Gestiona ventana, ciclo de vida de la aplicación y acceso
al sistema de archivos. No contiene lógica de negocio.

### Frontend (`frontend/`)

Presentación e interacción exclusivamente.

- Renderiza datos que recibe del backend
- Gestiona navegación y estado de vista
- Proyecta experiencias (Dashboard, Coach, Explainability, Journey)

**No hace:** procesar telemetría, calcular métricas, decidir foco, reinterpretar
dimensiones de Attribution cuando el motor falla.

### Backend (`backend/`)

Frontera entre el mundo exterior y el Core.

- Expone endpoints HTTP y valida entradas
- Lee y escribe en SQLite
- Llama al Core y serializa sus resultados
- Gestiona ficheros subidos y errores de transporte

**No hace:** analizar telemetría ni generar recomendaciones de dominio.
Lecturas GET de training **no** persisten perfil (ADR-013).

### Core (`core/`)

Toda la lógica de dominio. Detallado en las secciones anteriores.

### Database (`database/`)

Ficheros SQLite y migraciones 0001–0010. Accedido únicamente desde el backend.
Diseño congelado: `DATABASE_DESIGN.md` v1.0.

### Project memory (`docs/project/`)

Memoria oficial del proyecto (PMS). Obligatorio actualizar al cerrar fases.

---

## Frontera entre backend y Core

El backend actúa como adaptador. Un endpoint típico sigue este patrón:

```
1. Recibe la petición HTTP y valida el payload
2. Lee de SQLite los datos necesarios
3. Llama al Core con estructuras Python puras
4. Recibe estructuras Python puras del Core
5. Persiste lo que deba persistirse
6. Serializa la respuesta a JSON
```

Los pasos 1, 2, 5 y 6 son del backend. El paso 3–4 es del Core. Esta frontera
no debe difuminarse: en cuanto el Core recibe un objeto de FastAPI o una sesión
de base de datos, deja de ser independiente.

---

## Estructura completa del proyecto

```
DDP/
├── core/          Motores de dominio independientes (Python)
├── backend/       API FastAPI — adaptador HTTP sobre el Core
├── frontend/      React + Tauri — proyección
├── database/      SQLite y migraciones
├── docs/          Documentación del proyecto
│   └── project/   Project Memory System (PMS) + PMS_SPEC
├── tools/         Infraestructura de desarrollo
│   └── pms/       Project Memory Infrastructure (PMI v1)
└── assets/        Recursos compartidos (p. ej. training YAML)
```

---

## Estado actual

| Capa | Estado |
|------|--------|
| Frontend | Operativo — Dashboard A, Coach B, Explainability C, Journey D, Telemetría, Sesiones, Settings |
| Backend | Operativo — import, analysis, interpretation, attribution, strategy, training, active driver |
| Core | Operativo — telemetry, engineer, attribution, development_strategy, training; reports weekly/career stubs |
| Database | Migraciones 0001–0010 · diseño congelado |
| Tauri | Configurado · shell de escritorio |
| PMS | Memoria oficial en `docs/project/` |
| PMI | **v1** · `tools/pms/` · `/close-phase` |
| AICS | **v2.1** · GitHub docs + prompt auto desde `PROJECT_CONFIGURATION` |

Producto: **DDP 4.0 Release Candidate** (post RCA-1 + PMI + AICS v2.1).  
Detalle vivo: `docs/project/PROJECT_STATE.md`.  
Relievo: `docs/project/HANDOVER.md`.  
Entrada IA: `docs/project/AI_BOOTSTRAP.md`.  
Config: `docs/project/PROJECT_CONFIGURATION.md`.  
Spec: `docs/project/PMS_SPEC.md`.

Documentos de motor congelados (no modificar sin descongelación explícita):

- `IMPORT_PIPELINE.md` v1.0
- `ANALYSIS_ENGINE.md` v1.0
- `INTERPRETATION_ENGINE.md` v1.0
- `DATA_MODEL.md` / `DATABASE_DESIGN.md` v1.0

---

## Flujo de mantenimiento del PMS (oficial)

El mantenimiento ya no es solo manual. Procedimiento:

```bash
py -m tools.pms.close_phase --phase "…"
```

1. Desarrollar respetando filosofía, arquitectura y ADRs.
2. Tests.
3. `validate_pms`.
4. Actualizar PMS (STATE, HANDOVER, SESSION_NOTES, …).
5. ADR / DECISIONS si proceden (append-only).
6. AICS: actualizar `AI_BOOTSTRAP.md` si cambió el contexto general.
7. `sync_docs`.
8. Validación final → cerrar o permanecer ABIERTA.

Detalle: `docs/project/PMS_SPEC.md` · checklist:
`docs/project/PHASE_CLOSE_CHECKLIST.md`.

---

*Documento vivo. Actualizar cuando cambie la estructura de capas, el pipeline
de motores o el contrato entre ellas. El estado operativo detallado vive en
el PMS (`PROJECT_STATE.md`).*
