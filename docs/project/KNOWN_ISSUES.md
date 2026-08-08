# DDP — Known Issues

> Solo incidencias **abiertas**.
> Cuando se resuelva una incidencia → **eliminarla** de este documento.
> No es un changelog. Para historia funcional: [`CHANGELOG_AI.md`](CHANGELOG_AI.md).
>
> Parte del **Project Memory System (PMS v1)**.

**Última revisión:** 2026-08-08  
**Fuente principal:** `docs/AUDIT_REPORT_RCA1.md` § Problemas pendientes

---

## KI-001 — Ficheros `.ibt` huérfanos en disco tras delete

| Campo | Valor |
|-------|-------|
| **Prioridad** | MEDIO |
| **Descripción** | `delete_session` elimina filas de BD pero puede dejar archivos de telemetría en el almacén gestionado. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `backend/importing/service.py` (delete / storage), almacén de telemetría en disco |

---

## KI-002 — Política de `analysis_report` tras borrar sesión

| Campo | Valor |
|-------|-------|
| **Prioridad** | MEDIO |
| **Descripción** | Tras delete, pueden quedar informes de interpretación / narrativa huérfanos o desanclados. Falta decisión de producto: detach vs delete en cascada. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | Repositorios de `analysis_report`, flujo `delete_session` |

---

## KI-003 — Helpers legacy sin callers calientes

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | `evolutionCoach.ts` y `composeCoachBrief.ts` (deprecated) permanecen documentados; no borrar hasta pase de dead-code confirmado. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `frontend/src/pages/dashboard/evolutionCoach.ts`, `composeCoachBrief.ts` |

---

## KI-004 — React Query instalado sin uso real de `useQuery`

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Provider presente; invalidación actual vía event bus `ddp:product-mutated`. Decidir adoptar React Query de verdad o retirar la dependencia. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | Frontend providers / `frontend/src/lib/productSync.ts` |

---

## KI-005 — Stubs Core de reports weekly / career

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Módulos `core/reports/weekly.py` y `career.py` existen como stubs; Informes fuera del nav RC. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `core/reports/weekly.py`, `core/reports/career.py` |

---

## KI-006 — Espejo `sessionContext` TypeScript / Python

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Riesgo de drift entre clasificación de tipo de sesión en Core y tipado/espejo en frontend. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `core/training/session_context.py`, espejo TS en frontend |

---

## KI-007 — Sin endpoint dedicado «eliminar perfil»

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Hoy la limpieza de perfil es parcial vía remove-all / reset DDP. Falta API de producto dedicada. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `backend/api/training_routes.py`, reset / profile services |

---

## KI-008 — Discard historial importa solo ~200 batches

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Edge case: el historial de discard/import puede truncar a ~200 batches antiguos. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | Lógica de import log / discard |

---

*Revisar al cerrar cada fase. Resolver → borrar la entrada.*
