# DDP 4.0 — Release Candidate Audit (RCA-1)

**Fecha:** 2026-08-08  
**Alcance:** Auditoría integral de producto (no nuevas features)  
**Filosofía:** DDP no analiza telemetrías. DDP desarrolla pilotos.  
**Criterio de cierre:** Base sólida, coherente y estable para la siguiente etapa (Coach/Ingeniero de pista en tiempo real).

---

## Resumen ejecutivo

Se auditaron flujos, sincronización, datos, Coach, UX, consistencia, arquitectura y código.  
**150 tests pytest pasan** (144 previos + 6 de contratos RCA-1).

Se corrigieron los fallos que **rompían coherencia de perfil o la historia del piloto**:

1. Los GET de coach **escribían el perfil** y podían deshacer «Quitar del perfil».
2. Borrar una sesión **no reconstruía** la cronología (EMA contaminada).
3. Rebuild vacío **dejaba career/índices** huérfanos.
4. Dashboard/Journey **mezclaban multi-piloto**; no había cambio de piloto usable.
5. El Dashboard **reinterpretaba** dimensiones en cliente cuando Attribution fallaba.
6. Narrativa Strategy usaba **jerga de motor**.
7. Informes en nav era un **stub** que generaba expectativa falsa.
8. Tras mutaciones en Telemetría, Dashboard/Journey podían quedar **obsoletos**.

DDP queda en estado **Release Candidate usable** para preparar el Coach en tiempo real. Quedan deudas BAJO/MEDIO documentadas (sin maquillaje, sin refactor cosmético).

---

## Arquitectura revisada

Pipeline oficial (intacta):

```
Import → Analysis → Attribution → Strategy Facts → Development Strategy
       → Planner → Coach → Dashboard → Journey
```

| Capa | Rol |
|------|-----|
| `core/` | Motores de dominio (sin HTTP/SQLite/UI) |
| `backend/` | Orquestación, persistencia, API |
| `frontend/` | Proyección (no debe decidir foco) |
| `database/` | SQLite + migraciones 0001–0010 |

**Violación corregida:** lectura HTTP que mutaba perfil; interpretación local de dimensiones en Dashboard.  
**Violación documentada (no eliminada aún):** helpers legacy en `evolutionCoach.ts`; espejo TS de `sessionContext`; `ARCHITECTURE.md` § Estado actual obsoleto.

---

## Motores revisados

| Motor | Estado RCA-1 |
|-------|----------------|
| Import | OK — dedup por hash; delete ahora rebuild cronología |
| Analysis | OK — sin cambios; fases congeladas respetadas |
| Interpretation | OK — sin cambios de motor |
| Attribution | OK — on-demand; inputs = perfil persistido |
| Strategy Facts | OK — hidrata desde SQLite actual |
| Development Strategy | OK — prosa humanizada (sin jerga de retorno/jerarquía) |
| Planner / Coach training | OK — persist solo vía POST incorporate / rebuild / remove |
| Dashboard / Journey | OK — filtran por piloto activo; invalidación por evento |

---

## Frontend revisado

| Pantalla | Hallazgo | Acción |
|----------|----------|--------|
| Dashboard | GET mutaba perfil; reinterpretaba dims; multi-piloto | Corregido |
| Trayectoria | Mezcla multi-piloto con Strategy de uno | Corregido |
| Telemetría | Incorporaba vía GET `/skills` | POST `/profile/incorporate` |
| Sesiones | GET plan (ahora read-only) | OK |
| Settings | Sin cambio de piloto | Selector + API active |
| Informes | Stub en nav | Fuera del nav RC |
| UI Gallery | Solo DEV | Sin incidencias |

---

## Backend revisado

| Área | Hallazgo | Acción |
|------|----------|--------|
| `GET …/training-plan\|skills\|progress` | `persist=True` por defecto | Default `False` |
| Incorporación | No había POST explícito | `POST …/profile/incorporate` |
| `delete_session` | Cleanup sin rebuild | Rebuild con `exclude_session_id` |
| `rebuild_profile_chronology` | Vacío no limpiaba career | Wipe indexes + career |
| Supersede en rebuild | Todos los objetivos activos | Solo `training_assignment` |
| Piloto activo | Solo DB/import | `GET/PUT /drivers/active`, `GET /drivers` |

---

## Flujos revisados

| Flujo | Resultado |
|-------|-----------|
| Importación | OK |
| Importaciones repetidas (hash) | OK |
| Importaciones cronológicas / out-of-order | OK (rebuild en incorporate) |
| Recalcular / Añadir al perfil | OK vía POST incorporate |
| Quitar del perfil | OK + ya no se reincorpora al abrir Dashboard |
| Eliminar telemetría | OK + rebuild cronología |
| Eliminar perfil (dedicado) | No existe endpoint; parcial vía remove-all / reset — documentado |
| Reset DDP | OK |
| Cambio de piloto | Completado (Settings + API) |
| Dashboard / Journey / Coach / Strategy / Planner / Settings | Coherentes con piloto activo |

---

## Problemas encontrados

### CRÍTICO

| ID | Problema | Por qué | Corrección |
|----|----------|---------|------------|
| C1 | GET coach persistía perfil | Abrir Dashboard/Sesiones reincorporaba tras «quitar» | `persist=False` + POST incorporate |
| C2 | Multi-piloto sin piloto activo en UI | Journey/Dashboard mezclaban biografías | Active driver API + filtro loaders + Settings |
| C3 | Journey hitos globales vs Strategy de un driver | Dos historias distintas | Filtrar sesiones por piloto foco |

### ALTO

| ID | Problema | Por qué | Corrección |
|----|----------|---------|------------|
| A1 | `delete_session` sin rebuild | EMA/skills de sesiones posteriores contaminadas | Rebuild tras cleanup |
| A2 | Rebuild vacío dejaba career | Fortalezas/debilidades fantasma | Clear career + wipe indexes |
| A3 | Dashboard reinterpretaba Attribution | UI decide causalidad | Estado «insuficiente» |
| A4 | Sin invalidación cross-view | Datos viejos tras mutar Telemetría | `ddp:product-mutated` |

### MEDIO

| ID | Problema | Por qué | Estado |
|----|----------|---------|--------|
| M1 | Informes stub en nav | Expectativa falsa | Fuera del nav |
| M2 | Jerga Strategy («retorno», «jerarquía») | Suena a motor | Reescrita |
| M3 | Rebuild supersedía todos los objetivos | Podía tumbar no-training | Solo training_assignment |
| M4 | Ficheros `.ibt` huérfanos en disco tras delete | Solo se borran filas | **Pendiente** |
| M5 | `analysis_report` sin sesiones tras delete | Narrativa huérfana posible | **Pendiente** (diseño) |

### BAJO

| ID | Problema | Estado |
|----|----------|--------|
| B1 | `evolutionCoach` legacy sin callers calientes | Documentado; no borrar en RCA |
| B2 | `composeCoachBrief.ts` deprecated sin imports | Documentado |
| B3 | React Query instalado sin `useQuery` | Mitigado con event bus; deuda |
| B4 | `core/reports/weekly|career` stubs | Documentado |
| B5 | Espejo `sessionContext` TS/Python | Riesgo de drift; documentado |
| B6 | `ARCHITECTURE.md` § Estado actual obsoleto | **Pendiente** actualizar doc |
| B7 | Discard historial importa solo ~200 batches | Edge case antiguo |

---

## Archivos afectados (correcciones)

**Backend**
- `backend/training/service.py`
- `backend/api/training_routes.py`
- `backend/importing/service.py`
- `backend/training/tests/test_rca1_contracts.py` *(nuevo)*

**Core**
- `core/development_strategy/narrative.py`
- `core/development_strategy/tests/test_edges.py`
- `core/development_strategy/tests/test_engine.py`
- `core/development_strategy/tests/test_rca1_narrative_tone.py` *(nuevo)*

**Frontend**
- `frontend/src/services/api.ts`
- `frontend/src/lib/productSync.ts` *(nuevo)*
- `frontend/src/pages/Telemetry.tsx`
- `frontend/src/pages/Dashboard.tsx`
- `frontend/src/pages/Driver.tsx`
- `frontend/src/pages/Settings.tsx`
- `frontend/src/pages/Reports.tsx`
- `frontend/src/pages/dashboard/loadDashboard.ts`
- `frontend/src/pages/driver/loadDriverJourney.ts`
- `frontend/src/theme/navigation.ts`

---

## Correcciones realizadas

1. **Lectura ≠ escritura en coach** — GET no persiste; POST `/sessions/{id}/profile/incorporate` es la vía explícita.
2. **Delete = remove + hard delete** — rebuild cronológico antes del borrado duro.
3. **Perfil vacío limpio** — sin career/índices fantasma.
4. **Piloto activo de producto** — listar/fijar en Settings; Dashboard/Journey filtran.
5. **UI proyecta, no interpreta** — dimensiones insuficientes sin fallback local.
6. **Coach Strategy en lenguaje de pista**.
7. **Invalidación** — evento `ddp:product-mutated` tras mutaciones.
8. **Informes** — fuera del nav RC.
9. **Tests de contrato** RCA-1 añadidos; suite completa en verde.

---

## Problemas pendientes

| Prioridad | Ítem | Notas |
|-----------|------|-------|
| MEDIO | Purge disco en `delete_session` | Evitar huérfanos `.ibt` |
| MEDIO | Política de `analysis_report` tras delete | Detach vs delete |
| BAJO | Limpiar `evolutionCoach` legacy / `composeCoachBrief.ts` | Solo tras confirmación de cero callers |
| BAJO | Actualizar `docs/ARCHITECTURE.md` § Estado actual | Doc stale, no código |
| BAJO | Adoptar React Query de verdad o quitar provider | Ya hay event bus mínimo |
| BAJO | Endpoint «eliminar perfil» dedicado | Hoy: reset o quitar sesiones |
| — | Coach tiempo real | **Siguiente etapa** (fuera de RCA-1) |

---

## Recomendaciones

1. Antes del Coach en tiempo real: ejercitar E2E manual  
   import → incorporar → Dashboard → quitar → Dashboard (no debe volver) → borrar → Journey.
2. Mantener la regla: **ningún GET escribe perfil**.
3. No reactivar interpretación local en Dashboard; si Attribution falla, decirlo.
4. Actualizar `ARCHITECTURE.md` en un PR documental aparte.
5. No borrar helpers legacy hasta un pase de dead-code dedicado (este informe los lista).

---

## Riesgos

| Riesgo | Mitigación |
|--------|------------|
| Piloto activo vacío tras reset | Fallback a última sesión |
| Sesión externa importada con otro piloto activo | No aparece hasta cambiar piloto (intencional) |
| Plan efímero en Dashboard si práctica no está en perfil | Ya no se auto-incorpora; estado más honesto |
| Drift `sessionContext` TS/Python | Tests de clasificación existentes; vigilar en RC |

---

## Elementos revisados sin incidencias

- Deduplicación de import por `content_hash`
- Orden cronológico por `session.started_at` (no orden de importación)
- `remove_session_from_profile` + rebuild (ya correcto antes de RCA)
- Out-of-order / perfil contaminado → rebuild en incorporate
- Dual timeline development vs competitive
- Strategy **no persistida** (siempre recomputada)
- Priority shift Dashboard solo desde Strategy `action === "change"`
- Foco Dashboard desde Strategy usable (no `weaknesses[0]`)
- Reset DDP coherente (tablas usuario + telemetría disco)
- Cascades FK sesión → laps/analysis (esquema)
- Integrity helpers / fase scripts
- Fases congeladas Import/Analysis/Interpretation **no rotas**

---

## Testing (Fase 9)

```text
py -m pytest
150 passed
```

Nuevos:
- `backend/training/tests/test_rca1_contracts.py`
- `core/development_strategy/tests/test_rca1_narrative_tone.py`

Frontend lint no ejecutado en este entorno (`node` no resoluble en PATH del shell). Recomendado: `npm run lint` / `npm run build` en la máquina de desarrollo.

---

## Criterio de cierre RCA-1

**Cumplido para estabilización de producto offline:**  
los flujos de perfil ya no dejan estados imposibles, la UI cuenta una sola historia por piloto activo, y los GET dejan de mutar el programa del piloto.

**No incluido (siguiente gran etapa):** Coach/Ingeniero de pista en tiempo real.

---

*Fin del informe RCA-1.*
