# DDP — Project Context

> Documento de incorporación al proyecto.
> Debe permitir comprender DDP en menos de cinco minutos.
>
> Parte del **Project Memory System (PMS v1)**.
> Lectura obligatoria al inicio de cualquier conversación nueva
> (después de `DDP_PHILOSOPHY.md` y `ARCHITECTURE.md`).

**Última actualización:** 2026-08-08

---

## Qué es DDP

**Driver Development Program** — sistema de desarrollo de pilotos de simracing
(aplicación de escritorio, Tauri + React + FastAPI + Core Python + SQLite).

No es un visor de telemetrías ni un generador de informes.
La telemetría es la materia prima; el producto es el **piloto que mejora**.

Lema:

> **No analizamos vueltas. Desarrollamos pilotos.**

Detalle: [`docs/DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md).

---

## Filosofía

1. La telemetría es un medio, nunca el objetivo.
2. Cada sesión es un capítulo de una historia mayor.
3. Entrenar con propósito > acumular datos.
4. Toda recomendación debe poder justificarse con evidencia.
5. El Coach enseña; no solo analiza.
6. Pensar en el piloto de dentro de seis meses.

**Confianza** = evidencia + explicación. Sin eso, no hay desarrollo real.

---

## Arquitectura general

Cuatro capas con dependencias hacia dentro:

```text
Tauri (shell)
  → Frontend (React · proyección, nunca decide)
    → Backend (FastAPI · orquestación / persistencia)
      → Core (motores deterministas · dominio)
        → SQLite
```

Pipeline oficial de producto (DDP 4.0):

```text
Import → Analysis → Interpretation
       → Attribution → Strategy Facts → Development Strategy
       → Planner → Coach → Dashboard → Journey
```

Detalle: [`docs/ARCHITECTURE.md`](../ARCHITECTURE.md).

---

## Motores existentes

| Motor | Pregunta | Estado |
|-------|----------|--------|
| **Import Pipeline** | ¿Qué hechos entran? | Implementado · doc v1.0 congelada |
| **Analysis Engine** | ¿Qué ocurrió? | Implementado · doc v1.0 congelada |
| **Interpretation Engine** | ¿Qué problema priorizar en esta sesión? | Implementado · doc v1.0 congelada |
| **Performance Attribution** | ¿Por qué ocurrió? | Implementado · consumido por Dashboard/Coach |
| **Strategy Facts** | Hechos estables para Strategy | Implementado · hidrata desde SQLite |
| **Development Strategy** | ¿Qué debo hacer ahora? | Implementado · on-demand, sin persistir |
| **Training Planner** | ¿Cómo entreno hoy? | Implementado · assignments / ejercicios |
| **Progress / Profile** | ¿Ha funcionado? ¿Cómo evoluciono? | Implementado · dual timeline |
| **Coach / Dashboard / Journey** | ¿Cómo se lo digo al piloto? | Proyección UI · Fases A–D + Narrative Intelligence |

---

## Qué está congelado

No modificar sin proceso explícito de descongelación:

| Ámbito | Documento / alcance |
|--------|---------------------|
| Arquitectura de capas | `ARCHITECTURE.md` (contrato Core) |
| Modelo de datos | `DATA_MODEL.md` v1.0 |
| Diseño BD | `DATABASE_DESIGN.md` v1.0 · migraciones 0001–0010 |
| Import | `IMPORT_PIPELINE.md` v1.0 |
| Analysis | `ANALYSIS_ENGINE.md` v1.0 |
| Interpretation | `INTERPRETATION_ENGINE.md` v1.0 |
| Driver Experience A–C | Dashboard · Coach briefing · Explainability (proyección) |
| Fase D Journey | Memoria deportiva (`/driver`) — entregada; no reabrir sin motivo |

Los motores de dominio **no** se reescriben desde la UI.
La UI proyecta; no calcula foco ni causalidad.

---

## Documentos obligatorios (PMS)

| Documento | Rol |
|-----------|-----|
| [`BOOTSTRAP.md`](BOOTSTRAP.md) | Entrada humana (AICS) |
| [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) | Entrada IA + protocolo + prompt ChatGPT |
| [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md) | Config global (repos/URLs) |
| [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) | Incorporación (este) |
| [`PROJECT_STATE.md`](PROJECT_STATE.md) | Estado actual visual |
| [`DECISIONS.md`](DECISIONS.md) | Decisiones de producto |
| [`ADR.md`](ADR.md) | Decisiones técnicas |
| [`ROADMAP.md`](ROADMAP.md) | Evolución de producto |
| [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md) | Incidencias abiertas |
| [`HANDOVER.md`](HANDOVER.md) | Relievo de fase |
| [`SESSION_NOTES.md`](SESSION_NOTES.md) | Diario cronológico |
| [`CHANGELOG_AI.md`](CHANGELOG_AI.md) | Cambios funcionales |
| [`AI_GUIDE.md`](AI_GUIDE.md) | Papel de la IA en DDP |
| [`PMS_SPEC.md`](PMS_SPEC.md) | Especificación oficial PMS + PMI + AICS |
| [`PHASE_CLOSE_CHECKLIST.md`](PHASE_CLOSE_CHECKLIST.md) | Checklist de cierre |

**Entrada:** humanos → `BOOTSTRAP.md` · IA → `AI_BOOTSTRAP.md`  
**Infraestructura:** `tools/pms/` — procedimiento `/close-phase`  
**Norte de producto:** `docs/DDP_PHILOSOPHY.md`  
**Reglas de ingeniería:** `docs/DDP_RULES.md`  
**Coach:** `docs/COACH_PHILOSOPHY.md`

---

## Estado general

| Campo | Valor |
|-------|-------|
| Versión de producto | **DDP 4.0** |
| Estado | **Release Candidate** (post RCA-1) |
| Tests | 150 pytest en verde (auditoría RCA-1) |
| Última fase cerrada | AICS v2 (protocolo + ChatGPT Startup Prompt) |
| Próxima gran fase | **Coach en tiempo real / Ingeniero de pista** |

El producto offline es usable: importar → analizar → incorporar al perfil →
Dashboard / Coach / Explainability / Trayectoria con una sola historia por
piloto activo.

---

## Próxima gran fase

```text
Coach en tiempo real
        ↓
Ingeniero de pista
        ↓
IA conversacional (voz de DDP, nunca cerebro)
        ↓
Mentor deportivo
```

La IA **comunica, explica, enseña, resume, conversa y acompaña**.
Nunca decide, interpreta telemetría, genera hechos ni modifica Attribution
o Strategy. Ver [`AI_GUIDE.md`](AI_GUIDE.md).

---

## Orden de lectura al incorporar

**Primero:** [`BOOTSTRAP.md`](BOOTSTRAP.md) (humano) o
[`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) (IA).

Luego (profundidad):

1. `docs/DDP_PHILOSOPHY.md`
2. `docs/ARCHITECTURE.md`
3. `docs/project/PROJECT_CONTEXT.md` ← estás aquí
4. `docs/project/PROJECT_STATE.md`
5. `docs/project/DECISIONS.md`
6. `docs/project/ADR.md`
7. `docs/project/HANDOVER.md`
8. `docs/project/KNOWN_ISSUES.md`

Solo después: desarrollo.

---

*PMS v1 · memoria oficial del proyecto.*
