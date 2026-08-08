# DDP — Bootstrap (humanos)

> Punto oficial de entrada para cualquier **desarrollador humano**.
>
> Las Inteligencias Artificiales deben empezar por
> [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) (AI Context System).
>
> Parte del **Project Memory System (PMS)** + **AICS v2.1**.

**Última actualización:** 2026-08-08

---

## Qué es el PMS

El **Project Memory System** es la memoria oficial de DDP.

Vive en `docs/project/`. No es documentación opcional.

Es la **única fuente oficial de verdad** del estado, las decisiones y el
historial del proyecto. Los chats (Cursor, ChatGPT u otros) **no** son
fuente de verdad.

La infraestructura que lo valida y cierra fases es la **PMI**
(`tools/pms/`, procedimiento `/close-phase`).

Especificación: [`PMS_SPEC.md`](PMS_SPEC.md).  
**Única fuente válida** de URL del repositorio, rama, rutas oficiales y
configuración AICS: [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md).
Nunca escribir esa URL a mano en otros documentos.

---

## AI Context System (AICS)

| Entrada | Para quién |
|---------|------------|
| **Este documento** (`BOOTSTRAP.md`) | Humanos |
| [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) | Inteligencias Artificiales |

El AICS **no sustituye** al PMS. Es la **puerta de entrada** al PMS.

Incluye:

- **AI INITIALIZATION PROTOCOL** (obligatorio para toda IA)
- **ChatGPT Startup Prompt** (prompt oficial generado desde
  `PROJECT_CONFIGURATION.md` — nunca editar la URL a mano)

---

## Documentos del PMS

| Documento | Propósito | Cuándo leerlo |
|-----------|-----------|---------------|
| [`BOOTSTRAP.md`](BOOTSTRAP.md) | Entrada humana al PMS | Al incorporar un desarrollador |
| [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) | Entrada IA + protocolo + prompt ChatGPT | Al iniciar cualquier conversación con IA |
| [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md) | Configuración global (única fuente de URLs/repos) | Al cambiar repo de docs o regenerar prompts |
| [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) | Incorporación &lt; 5 min | Tras filosofía/arquitectura |
| [`PROJECT_STATE.md`](PROJECT_STATE.md) | Estado actual visual | Siempre antes de desarrollar |
| [`DECISIONS.md`](DECISIONS.md) | Decisiones de producto | Antes de cambiar comportamiento de producto |
| [`ADR.md`](ADR.md) | Decisiones técnicas | Antes de tocar arquitectura/motores/contratos |
| [`ROADMAP.md`](ROADMAP.md) | Evolución de producto | Para ubicar “qué sigue” |
| [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md) | Incidencias abiertas | Antes de tocar deuda o delete/perfil |
| [`HANDOVER.md`](HANDOVER.md) | Relievo de fase | Al retomar trabajo / nueva sesión |
| [`SESSION_NOTES.md`](SESSION_NOTES.md) | Diario cronológico | Para historia de fases |
| [`CHANGELOG_AI.md`](CHANGELOG_AI.md) | Cambios funcionales | Para entender el producto percibido |
| [`AI_GUIDE.md`](AI_GUIDE.md) | Papel de la IA | Antes de proponer features de IA |
| [`PMS_SPEC.md`](PMS_SPEC.md) | Spec PMS/PMI/AICS | Al cambiar infra de memoria |
| [`PHASE_CLOSE_CHECKLIST.md`](PHASE_CLOSE_CHECKLIST.md) | Checklist de cierre | Al cerrar una fase |

Norte de producto (fuera de `project/` pero obligatorio):

| Documento | Propósito |
|-----------|-----------|
| [`../DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md) | Qué es DDP y qué merece existir |
| [`../ARCHITECTURE.md`](../ARCHITECTURE.md) | Capas, pipeline, contrato Core |

---

## Orden oficial de lectura

Antes de desarrollo importante:

1. `docs/DDP_PHILOSOPHY.md`
2. `docs/ARCHITECTURE.md`
3. `docs/project/PROJECT_CONTEXT.md`
4. `docs/project/PROJECT_STATE.md`
5. `docs/project/DECISIONS.md`
6. `docs/project/ADR.md`
7. `docs/project/HANDOVER.md`
8. `docs/project/KNOWN_ISSUES.md`

**Atajo para humanos nuevos:** este `BOOTSTRAP.md` → luego el orden anterior.  
**Atajo para IA:** `AI_BOOTSTRAP.md` → profundizar solo con enlaces del PMS.

---

## Prioridad en caso de contradicción

```text
DDP_PHILOSOPHY
      ↓
ARCHITECTURE
      ↓
ADR
      ↓
DECISIONS
      ↓
PROJECT_STATE
      ↓
HANDOVER
```

1. La **misión** manda sobre la implementación.
2. La **arquitectura** manda sobre atajos de capa.
3. Los **ADR** mandan sobre detalles técnicos ad hoc.
4. Las **DECISIONS** de producto mandan sobre preferencias locales.
5. **STATE** y **HANDOVER** describen el presente operativo; si chocan con
   ADR/DECISIONS/filosofía, se corrigen STATE/HANDOVER (no al revés).

Los chats nunca ganan a esta cadena.

---

## Consulta rápida según necesidad

| Necesitas… | Consulta |
|------------|----------|
| Entender qué es DDP | `DDP_PHILOSOPHY.md` |
| Ver capas y pipeline | `ARCHITECTURE.md` |
| Estado “ahora” | `PROJECT_STATE.md` + `HANDOVER.md` |
| Si puedo tocar un motor | `ADR.md` + docs congelados del motor |
| Por qué el foco es X | `DECISIONS.md` + Strategy docs |
| Qué está roto | `KNOWN_ISSUES.md` |
| Papel de la IA | `AI_GUIDE.md` |
| Cerrar una fase | `/close-phase` · `PMS_SPEC.md` · checklist |

---

## Cierre de fase

```bash
py -m tools.pms.validate_pms
py -m tools.pms.close_phase --phase "Nombre"
```

Tras cambios de contexto general, `/close-phase` regenera
`AI_BOOTSTRAP.md` y el **ChatGPT Startup Prompt** desde
`PROJECT_CONFIGURATION.md`.

`BOOTSTRAP.md` solo cambia si cambia la **estructura** del PMS.  
Ninguna URL de documentación se duplica fuera de `PROJECT_CONFIGURATION.md`.

---

## Regla de oro

> El conocimiento vive en el proyecto. Nunca en los chats.

El PMS es la única fuente oficial de verdad.

---

*AICS v2.1 · puerta humana al PMS.*
