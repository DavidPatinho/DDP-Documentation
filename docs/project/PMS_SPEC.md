# DDP — Project Memory System Specification

> **Especificación oficial del PMS y de la Project Memory Infrastructure (PMI v1).**
>
> Este documento define qué es el PMS, qué documentos existen, quién puede
> modificarlos, cuándo se actualizan, qué formato tienen, qué validaciones
> pasan, qué scripts los mantienen y cómo se relacionan entre sí.
>
> Ante conflicto operativo de memoria del proyecto, manda este documento
> junto con `HANDOVER.md` / `PROJECT_STATE.md`.
> Ante conflicto de *misión de producto*, manda `DDP_PHILOSOPHY.md`.

**Versión:** PMI v1 + AICS v2.1 · 2026-08-08  
**Estado:** Vigente — infraestructura oficial de desarrollo  
**Comando de cierre:** `/close-phase`

---

## 1. Qué es el PMS

El **Project Memory System** es la memoria oficial de DDP.

Vive en `docs/project/`. No es documentación opcional ni un conjunto de notas.
Es la **única fuente oficial de verdad** del proyecto. Los chats no lo son.

La **Project Memory Infrastructure (PMI)** es la capa de herramientas en
`tools/pms/` que valida, actualiza y orquesta el mantenimiento del PMS.

El **AI Context System (AICS)** es la puerta de entrada al PMS:

| Entrada | Audiencia |
|---------|-----------|
| `BOOTSTRAP.md` | Desarrolladores humanos |
| `AI_BOOTSTRAP.md` | Inteligencias Artificiales |

El AICS **no sustituye** al PMS. Resume, enlaza, define el
**AI INITIALIZATION PROTOCOL** y genera el **ChatGPT Startup Prompt**.

```text
Development Infrastructure
        ↓
Project Memory System (PMS)
        ↓
AI Context System (AICS)
        ↓
Product Architecture
```

```text
docs/project/     →  PMS + AICS + PROJECT_CONFIGURATION
tools/pms/        →  PMI (scripts)
```

El producto runtime (`core/`, `backend/`, `frontend/`) **no** incluye la PMI.

---

## 1.1 PROJECT_CONFIGURATION.md

**Única fuente de configuración global** del AICS / PMS / PMI.

| Campo | Uso |
|-------|-----|
| `Project Name` | Nombre del proyecto |
| `Current Version` | Versión de producto |
| `Official Documentation Repository` | URL GitHub del repo público de documentación |
| `Repository Branch` | Rama (p. ej. `main`) |
| `Repository Docs Path` | Raíz de docs en ese repo |
| `AI Bootstrap Path` | Ruta a `AI_BOOTSTRAP.md` |
| `Bootstrap Path` | Ruta a `BOOTSTRAP.md` |
| `PMS Version` | Versión del sistema de memoria / AICS |

**Reglas:**

- Ningún otro documento escribe la URL (ni rama/rutas) a mano.
- El **ChatGPT Startup Prompt** se genera solo desde estos campos.
- Si cambia la URL → editar **solo** este archivo → regenerar prompt
  (`update_ai_bootstrap --refresh-prompt --apply` o `/close-phase`).
- `validate_pms` falla si la URL aparece duplicada fuera de este archivo
  y del bloque generado del prompt.

Mantenedor: humano (valores) + PMI (propagación al prompt).

---

## 1.2 AICS — BOOTSTRAP / AI_BOOTSTRAP

| Documento | Quién lo mantiene | Cuándo |
|-----------|-------------------|--------|
| `BOOTSTRAP.md` | Humano / agente | Solo si cambia la **estructura** del PMS |
| `AI_BOOTSTRAP.md` | Humano / agente + PMI | Si cambia contexto general; prompt siempre regenerable |

### AI INITIALIZATION PROTOCOL

Sección obligatoria dentro de `AI_BOOTSTRAP.md`. Define el procedimiento
que toda IA debe seguir antes de desarrollar (lectura de bootstrap, obtención
del repo desde `PROJECT_CONFIGURATION.md`, lectura PMS obligatoria, PMS &gt; chat).

### ChatGPT Startup Prompt

Sección obligatoria en `AI_BOOTSTRAP.md`, delimitada por:

```text
<!-- AICS:CHATGPT_PROMPT_BEGIN -->
…
<!-- AICS:CHATGPT_PROMPT_END -->
```

- **No** editar a mano el bloque.
- Generado por `tools/pms/update_ai_bootstrap.py` leyendo
  `PROJECT_CONFIGURATION.md`.
- Es el prompt oficial copy-paste para ChatGPT.

---

## 2. Documentos del PMS

| Documento | Propósito | Mutabilidad |
|-----------|-----------|-------------|
| `BOOTSTRAP.md` | Entrada humana (AICS) | Solo si cambia la **estructura** del PMS |
| `AI_BOOTSTRAP.md` | Entrada IA + protocolo + prompt ChatGPT | Contexto general + prompt auto |
| `PROJECT_CONFIGURATION.md` | Config global (repos/URLs) | Cuando cambie el repo oficial de docs |
| `PROJECT_CONTEXT.md` | Incorporación (&lt; 5 min) | Actualizar cuando cambie la visión/arquitectura general |
| `PROJECT_STATE.md` | Estado actual visual | **Cada cierre de fase** |
| `DECISIONS.md` | Decisiones de producto | **Append-only** (nunca reescribir vigentes) |
| `ADR.md` | Decisiones técnicas | **Append-only** (nunca reescribir aceptadas) |
| `ROADMAP.md` | Evolución de producto | Actualizar si cambia lo conseguido/siguiente/visión |
| `KNOWN_ISSUES.md` | Solo issues abiertos | Añadir / **eliminar al resolver** |
| `HANDOVER.md` | Relievo (una sola versión) | **Sobrescribir** en cada cierre |
| `SESSION_NOTES.md` | Diario cronológico | **Append-only** |
| `CHANGELOG_AI.md` | Cambios funcionales | **Append-only** (entradas nuevas) |
| `AI_GUIDE.md` | Papel de la IA | Actualizar si cambia la norma de IA |
| `PMS_SPEC.md` | Esta especificación | Actualizar cuando cambie la infra PMS/PMI/AICS |
| `PHASE_CLOSE_CHECKLIST.md` | Checklist reutilizable | Plantilla; copias por fase opcionales |

Documentos obligatorios incluyen siempre las puertas AICS + el núcleo PMS.

---

## 3. Quién puede modificarlos

| Actor | Permisos |
|-------|----------|
| Desarrollador / agente en fase | Actualizar PMS como parte del cierre |
| PMI (`tools/pms/*`) | Validar; append seguro; stamp de fechas; listar issues |
| Procedimientos futuros CI | Ejecutar `validate_pms` / `/close-phase` |
| Nadie | Reescribir en silencio decisiones/ADR congelados |
| Nadie | Usar chats como fuente de verdad frente al PMS |

**Regla:** DECISIONS y ADR solo crecen. Si una decisión se supersede, se marca
su `Estado` y se añade una nueva entrada — no se borra la historia.

---

## 4. Cuándo se actualizan

| Momento | Documentos |
|---------|------------|
| Durante desarrollo | Opcional: SESSION_NOTES parciales, issues nuevos |
| Antes de cerrar fase | STATE, HANDOVER, SESSION_NOTES (obligatorio) |
| Si hay decisión de producto | DECISIONS (nueva entrada) |
| Si hay decisión técnica | ADR (nueva entrada) |
| Si cambia el producto percibido | CHANGELOG_AI, ROADMAP |
| Si se abre/cierra incidencia | KNOWN_ISSUES |
| Si cambia contexto general | **`AI_BOOTSTRAP.md`** (filosofía, arquitectura, estado, ADR, decisiones, roadmap, próxima fase, issues críticos) + regenerar **ChatGPT Startup Prompt** |
| Si cambia `Official Documentation Repository` | Solo `PROJECT_CONFIGURATION.md` → regenerar prompt |
| Si cambia estructura del PMS | **`BOOTSTRAP.md`** |
| Tras `/close-phase` OK | Fase puede declararse CERRADA |

Sin actualización PMS → fase **ABIERTA**, aunque el código y los tests pasen.

---

## 5. Formatos

### DECISIONS
Cada entrada `## D-NNN — Título` con campos: Fecha, Título, Descripción,
Motivación, Impacto, Estado, Documentos, RFC.

### ADR
Cada entrada `## ADR-NNN — Título` con campos: Estado, Problema, Decisión,
Consecuencias.

### KNOWN_ISSUES
Solo abiertas. Entrada `## KI-NNN — Título` con Prioridad, Descripción,
Fecha, Estado, Archivos afectados. Al resolver → **eliminar** la entrada.

### SESSION_NOTES
Entradas `## YYYY-MM-DD — Título` con Objetivo, Trabajo, Problemas,
Decisiones, Resultado. Nunca sobrescribir historial.

### HANDOVER
Una sola versión viva. Secciones mínimas: última fase, qué se hizo, pendiente,
qué no tocar, próximo objetivo, estado general.

### PROJECT_STATE
Debe incluir versión, estado, motores, frontend/backend, última actualización
(fecha ISO).

---

## 6. Validaciones (`validate_pms`)

`py -m tools.pms.validate_pms` comprueba automáticamente:

| Check | Criterio |
|-------|----------|
| Documentos obligatorios | Existen en `docs/project/` |
| No vacíos | Documentos core con contenido real |
| Enlaces | Links relativos resolubles |
| PROJECT_STATE | Marcadores de estado + fecha |
| HANDOVER | Secciones mínimas |
| SESSION_NOTES | Al menos una entrada fechada |
| KNOWN_ISSUES | Formato válido |
| DECISIONS | Formato D-NNN + campos |
| ADR | Formato ADR-NNN + campos |
| PMS_SPEC | Presente y referencia cierre |

Exit code `0` = PASS · `1` = FAIL (fase abierta).

---

## 7. Scripts PMI (`tools/pms/`)

| Script | Responsabilidad |
|--------|-----------------|
| `close_phase.py` | Orquesta `/close-phase` (incluye AICS) |
| `validate_pms.py` | Informe de validación (incluye AICS) |
| `sync_docs.py` | Sync privado `docs/`; sonda repo público futuro |
| `publish_docs.py` | **Reservado** — publicación pública |
| `generate_handover.py` | Apoyo a generación de HANDOVER |
| `update_project_state.py` | Stamp / apoyo STATE |
| `update_changelog.py` | Append CHANGELOG_AI |
| `update_known_issues.py` | List / add / remove issues |
| `update_decisions.py` | Append decisiones (nunca reescribe) |
| `update_adr.py` | Append ADR (nunca reescribe) |
| `update_ai_bootstrap.py` | Mantener `AI_BOOTSTRAP.md` + regenerar ChatGPT Startup Prompt |
| `project_config.py` | Leer `PROJECT_CONFIGURATION.md` (única fuente de repo de docs) |
| `utils.py` | Rutas, constantes, reportes |

Detalle de uso: [`tools/README.md`](../../tools/README.md).

---

## 8. Relaciones entre documentos

```text
BOOTSTRAP (humanos) ──┐
AI_BOOTSTRAP (IA)   ──┼──► PMS (fuente de verdad)
                      │
DDP_PHILOSOPHY ──norte──► producto
ARCHITECTURE ──capas──► runtime + PMS/PMI/AICS
        │
        ▼
PROJECT_CONTEXT ──incorpora──► STATE / DECISIONS / ADR / HANDOVER
PROJECT_STATE   ◄──espejo──► HANDOVER (estado actual)
DECISIONS       ◄──producto──► ROADMAP / CHANGELOG_AI
ADR             ◄──técnico──► ARCHITECTURE / motores
KNOWN_ISSUES    ◄──deuda──► HANDOVER pendiente
SESSION_NOTES   ──diario──► (append; alimenta memoria)
AI_GUIDE        ──norma IA──► DECISIONS / ADR
PMS_SPEC        ──norma PMS──► tools/pms/*
```

### Entradas oficiales

| Quién | Empieza por |
|-------|-------------|
| Humano | `BOOTSTRAP.md` |
| IA | `AI_BOOTSTRAP.md` |

Orden de lectura profundo (desarrollo importante):

1. `DDP_PHILOSOPHY.md`
2. `ARCHITECTURE.md`
3. `PROJECT_CONTEXT.md`
4. `PROJECT_STATE.md`
5. `DECISIONS.md`
6. `ADR.md`
7. `HANDOVER.md`
8. `KNOWN_ISSUES.md`

Prioridad ante contradicción:

```text
PHILOSOPHY → ARCHITECTURE → ADR → DECISIONS → STATE → HANDOVER
```

---

## 9. Procedimiento oficial `/close-phase`

Concepto oficial de DDP (automatizable; no requiere slash-command del editor):

```text
1.  Desarrollar
2.  Ejecutar tests
3.  Ejecutar validación PMS          → validate_pms
4.  Actualizar documentación PMS
5.  Generar HANDOVER                 → generate_handover
6.  Revisar Issues                   → update_known_issues
7.  Actualizar PROJECT_STATE         → update_project_state
8.  Actualizar CHANGELOG             → update_changelog
9.  Actualizar ADR / DECISIONS       → update_adr / update_decisions
10. Mantener AICS                    → update_ai_bootstrap
      (AI_BOOTSTRAP si cambió contexto;
       ChatGPT Startup Prompt siempre regenerable desde PROJECT_CONFIGURATION)
11. Sincronizar documentación        → sync_docs
12. Validación final + Cerrar fase   → validate_pms PASS
```

Comando:

```bash
py -m tools.pms.close_phase --phase "Nombre de la fase"
```

Si cualquier paso falla → la fase permanece **ABIERTA**.

Checklist: [`PHASE_CLOSE_CHECKLIST.md`](PHASE_CLOSE_CHECKLIST.md).

---

## 10. Repositorio público de documentación (futuro)

Arquitectura preparada; **no implementada** en PMI v1:

```text
Repositorio privado (código + docs/)
        ↓  sync_docs (selección)
        ↓  publish_docs (publicación)
Repositorio público (solo documentación)
```

- Variable de entorno prevista: `DDP_PUBLIC_DOCS_REPO`
- Placeholder de ruta: `../ddp-docs`
- `publish_docs.py` está reservado y no publica todavía
- Ningún script debe asumir que el código de producto se publica

---

## 11. Integración con Git

| Práctica | Norma |
|----------|-------|
| Fuente de verdad | Repositorio privado |
| PMS en Git | Sí — `docs/project/` versionado |
| PMI en Git | Sí — `tools/pms/` versionado |
| Cierre de fase | Commit(s) que incluyen código **y** PMS actualizado |
| Hooks / CI futuros | Podrán ejecutar `validate_pms` como gate |

PMI v1 no instala hooks automáticamente; deja la puerta abierta.

---

## 12. Restricciones

La PMI / PMS **no**:

- modifica motores del Core
- modifica frontend de producto
- modifica backend funcional
- cambia la arquitectura runtime de DDP

Solo infraestructura de desarrollo y memoria del proyecto.

---

## 13. Confirmación normativa

A partir de PMI v1 + AICS v2.1:

1. El PMS es **infraestructura oficial** de DDP.
2. El AICS es la **puerta oficial** al PMS (humanos / IA).
3. `PROJECT_CONFIGURATION.md` es la **única fuente** de URL, rama y rutas AICS.
4. El **ChatGPT Startup Prompt** nunca se edita a mano; siempre se regenera.
5. ChatGPT usa el repositorio GitHub de documentación definido en la config.
6. El cierre de fase (`/close-phase`) regenera el prompt automáticamente.
7. El conocimiento vive en DDP, nunca en los chats.

---

*PMS_SPEC · PMI v1 · AICS v2.1 · referencia oficial.*
