# DDP 2.0 Architecture v1.1

## Capa ligera: Skill → Curvas del circuito → Ejercicio

> **Estado: vigente (v1.1)** — 2026-08-08  
> Sustituye la complejidad de v1.0 / RFC 001.  
> **RFC 002** — Simplificación del Driver Development Program.  
> **Implementación:** `core/training/circuit_skill_map.py` + `exercises.py` + YAML en `assets/training/`.  
> No rompe el núcleo existente.

### Gobernanza

| Aspecto | Regla |
|---------|-------|
| Documento anterior (v1.0) | **Superseded** — rolling weeks, career tables, fatigue budgets, full rebuild, etc. quedan fuera de alcance |
| Cambios futuros importantes | Proceso RFC |
| Núcleo intocable | Parser, importer, `SessionAnalysis`, Interpretation, APIs actuales, Dashboard existente |

---

## 1. Principio (todo el diseño cabe aquí)

```text
Análisis  →  Habilidad a mejorar  →  Curvas del circuito actual  →  Ejercicio
```

1. El análisis (vía Driver Coach existente) detecta **qué skill** entrenar.
2. Cada circuito tiene un **mapa** skill → curvas que mejor la entrenan.
3. El entrenador genera el ejercicio **anclando esa skill a esas curvas** del circuito de la sesión.

La habilidad es estable. Las curvas cambian con el circuito.

**Ejemplo — Control del acelerador (`throttle_control`)**

| Circuito | Curvas de entrenamiento |
|----------|-------------------------|
| Bathurst | Hell Corner, The Chase |
| Spa | La Source, Bruxelles |

Mismo skill. Distinto mapa. Mismo tipo de ejercicio, parametrizado por curvas.

---

## 2. Qué ya existe (no reinventar)

El bucle diario ya está:

```text
SessionAnalysis → Coach (skill) → Planner / Catalog (ejercicio) → Assignment → Progress → Profile
```

| Pieza | Rol actual | Qué falta |
|-------|------------|-----------|
| `core/training/coach.py` | Elige `primary_skill_id` | Nada estructural |
| `core/training/planner.py` | A veces usa `corner_ref` del finding con más pérdida | No usa un mapa pedagógico por circuito |
| `core/training/catalog.py` | Plantillas de ejercicio; acepta `corner_ref` | Debe aceptar **lista de curvas** del mapa |
| Catálogo `Corner` / `Track` en BD | Geometría / identidad de curvas | No hay relación skill ↔ curvas |
| Interpretation | Explica la sesión | Sigue fuera del entrenador |

**Conclusión:** no hace falta una capa `program/` grande ni un segundo producto. Solo falta el **mapa circuito↔skill↔curvas** y cablearlo al planner/catalog existentes.

---

## 3. Alcance de 2.0 (v1.1)

### En alcance

- Mapa estático (o semi-estático) por circuito: `skill_id → [corner_ref, …]`
- Resolución: sesión actual → `track_id` → mapa → curvas para la skill primaria
- Ejercicio generado con esas curvas (pasos, título, criterios anclados)
- Fallback si el circuito no tiene mapa o la skill no tiene curvas: comportamiento actual (finding de mayor pérdida / ejercicio genérico)

### Fuera de alcance (explícitamente)

- Planificación semanal / rolling 4+4
- Tablas nuevas de programa, career_event, roadmap de fases, fatigue budgets
- Rebuild híbrido / availability del piloto
- Módulos grandes nuevos (`core/program/…`)
- Cambios a parser, importer, SessionAnalysis, Interpretation, APIs/Dashboard actuales

Si en el futuro se quiere alguno de esos temas → RFC aparte, justificado.

---

## 4. Arquitectura

### 4.1 Flujo

```text
.ibt → Import → SessionAnalysis
                    │
                    ▼
              Driver Coach          (existente)
                    │ primary_skill_id
                    ▼
         Track Skill Map resolve    (NUEVO, pequeño)
                    │ corner_refs[] del circuito
                    ▼
         Training Planner/Catalog   (existente, ampliado)
                    │ TrainingExercise anclado a esas curvas
                    ▼
         Assignment / Progress / Profile  (existente)
```

Interpretation sigue en paralelo: **explica**, no decide el ejercicio.

### 4.2 Dónde vive el código (mínimo)

No crear paquete nuevo de producto. Extender `core/training/`:

| Pieza | Ubicación propuesta | Responsabilidad |
|-------|---------------------|-----------------|
| Mapa skill↔curvas | `core/training/track_skill_map.py` (+ datos YAML/JSON por track, o seed en repo) | Dado `track_key` + `skill_id` → `tuple[corner_ref, …]` |
| Cableado | `planner.py` / `catalog.py` | Pedir curvas al mapa; inyectarlas en el ejercicio |
| Opcional BD | Tabla ligera `track_skill_corner` **solo si** se quiere editar sin redeploy | Misma semántica que el seed |

Una función pública basta como contrato Core:

```text
resolve_training_corners(track_key, skill_id) -> tuple[str, ...]
```

Determinista. Sin I/O de red. Sin LLM.

### 4.3 Capas del sistema (sin cambios de forma)

```text
frontend → backend → core/training (+ mapa) → SQLite (catálogo track/corner ya existente)
```

---

## 5. Dominio — una sola idea nueva

### 5.1 `TrackSkillMap` (concepto)

Relación pedagógica:

```text
Track  ×  Skill  →  ordered Corner refs
```

Campos lógicos:

| Campo | Tipo | Notas |
|-------|------|-------|
| `track_key` | str | Identidad de circuito (slug / UUID / external id estable) |
| `skill_id` | str | Del catálogo cerrado existente |
| `corner_refs` | list[str] | Orden = preferencia de entrenamiento (1º = principal) |
| `notes` | str \| null | Opcional, para editores |

No es `TrackProfile` (estado del piloto en un combo).  
Es **conocimiento de pista**: qué curvas sirven para entrenar qué.

### 5.2 Relaciones

```text
Track 1 ── * TrackSkillMapEntry (por skill_id)
Corner  ←── corner_ref debe existir en el catálogo del Track

SessionAnalysis
  → Coach.primary_skill_id
  → Session.track → TrackSkillMapEntry
  → TrainingExercise.target_corners / target_corner_ref
```

### 5.3 Entidades que NO se añaden

No se introducen `training_program`, `training_week`, `driver_roadmap`, `career_event`, etc.  
El overlay actual de assignment/profile puede quedarse como está hasta que un RFC futuro justifique tablas propias.

---

## 6. Reglas del entrenador (simples)

1. **Skill primero.** La elige el Coach a partir de `SessionAnalysis` (como hoy).
2. **Curvas después.** Se resuelven con el mapa del circuito de la sesión.
3. **Ejercicio = plantilla de skill + curvas resueltas.**  
   Ejemplo de paso: *“En Hell Corner y The Chase, aplica el gas en un solo movimiento…”*
4. **Si hay mapa:** usar esas curvas aunque el peor `corner_finding` sea otra (el mapa es pedagógico; el finding sigue como evidencia/métrica si aplica).
5. **Si no hay mapa:** fallback al `corner_ref` del finding dominante o ejercicio sin curva nombrada (hoy).
6. **Misma skill en otro circuito:** mismas reglas de ejercicio, otras curvas.
7. **Determinismo:** mismo `track_key` + `skill_id` → mismas curvas; mismo analysis → mismo plan.

### 6.1 Evidencia vs pedagogía

| Fuente | Para qué sirve |
|--------|----------------|
| `corner_findings` / errores | Detectar skill e impacto; validar progreso |
| `TrackSkillMap` | Elegir **dónde entrenar** esa skill en este circuito |

No confundir “curva donde más pierdes hoy” con “curva donde mejor se entrena la skill”. Pueden coincidir; no es obligatorio.

---

## 7. Persistencia

**Preferida (simple):** datos versionados en el repo

```text
assets/track_skill_maps/{track_key}.yaml
# o un único maps.yaml
```

Ejemplo:

```yaml
track_key: bathurst
skills:
  throttle_control:
    - hell_corner
    - the_chase
  corner_exit:
    - forest_elbow
```

**Opcional más adelante:** tabla `track_skill_corner (track_id, skill_id, corner_id, sort_order)` si se necesita edición en producto. Misma semántica; no cambia el flujo.

No migrar el modelo de training completo. No abrir v1.1 de DATA_MODEL solo por esto salvo que se elija tabla.

---

## 8. Impacto por capa

| Capa | Impacto |
|------|---------|
| Core | Archivo pequeño de resolución + uso en planner/catalog |
| Backend | Casi ninguno: el pipeline de training ya llama a `plan_training`; pasa `track_key` si aún no llega limpio al Core |
| Frontend | Ninguno obligatorio; el ejercicio ya muestra pasos/título con nombres de curva |
| APIs | Sin breaking changes; el plan puede enriquecerse con `target_corners` en el JSON existente |
| Dashboard | Sin cambios de arquitectura |

---

## 9. Flujo de datos (detalle mínimo)

```text
1. plan_training(analysis, …)
2. primary_skill = coach / progress (existente)
3. track_key = analysis/session track ref
4. corners = resolve_training_corners(track_key, primary_skill)
5. exercise = exercise_for_skill(primary_skill, corners=corners, …)
6. assignment / profile como hoy
```

---

## 10. Riesgos (pocos)

| Riesgo | Mitigación |
|--------|------------|
| Circuito sin mapa | Fallback al comportamiento actual |
| `corner_ref` del mapa no coincide con catálogo | Validar en tests / seed contra corners del track |
| Mapa pedagógico vs peor curva del día confunde al piloto | Copy claro: “entrenamos X en estas curvas”; la métrica puede seguir el finding |
| Scope creep hacia programa semanal | Fuera de alcance; rechazar sin RFC |

---

## 11. Implementación (hecha)

1. `assets/training/track_knowledge.yaml` — Bathurst + Spa.  
2. `core/training/circuit_skill_map.py` — `resolve_training_corners`.  
3. `assets/training/exercise_templates/*.yaml` + `core/training/exercises.py`.  
4. `plan_training(..., track_key=)` + backend pasa el nombre del circuito.  
5. Coach: `priority = impact × confidence`.  

Nada más en esta entrega.

---

## 12. Criterio de hecho

- Misma skill en Bathurst y Spa produce ejercicios con **curvas distintas** del mapa.  
- Circuito sin mapa no rompe: plan igual que hoy.  
- Cero módulos grandes nuevos.  
- Cero planificación semanal.  
- Parser / importer / SessionAnalysis / Interpretation intactos.

---

## 13. Historial

| Versión | Fecha | Notas |
|---------|-------|-------|
| borrador | 2026-08-08 | Programa multi-semana complejo |
| v1.0 | 2026-08-08 | RFC 001 — rolling, tablas, roadmap, fatigue… |
| **v1.1** | 2026-08-08 | **RFC 002** — Simplificación: solo Análisis → Skill → Curvas → Ejercicio |

---

*Fin — DDP 2.0 Architecture v1.1 (simple).*
