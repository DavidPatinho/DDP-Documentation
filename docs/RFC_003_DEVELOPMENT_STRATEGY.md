# RFC 003 — Development Strategy Engine

| Campo | Valor |
|-------|-------|
| RFC | 003 |
| Título | Development Strategy Engine |
| Estado | **Aprobado** — listo para implementación |
| Fecha | 2026-08-08 |
| Autoría | Arquitectura DDP 3.0 |
| Sustituye / amplía | Amplía el bucle post-Attribution; no sustituye RFC 002 / DDP 2.0 |
| Documentos vinculados | `COACH_PHILOSOPHY.md`, `DEVELOPMENT_STRATEGY_ENGINE.md`, `PERFORMANCE_ATTRIBUTION_ENGINE.md`, `DDP_2_DEVELOPMENT_PROGRAM.md`, `ARCHITECTURE.md` |

---

## 1. Resumen

Motor Core con una sola pregunta:

> **¿Qué debo hacer ahora?**

Salida única `DevelopmentStrategy` (modelo simple), calculada **on-demand en
v1** (sin persistencia).

Principios rectores (`COACH_PHILOSOPHY.md`):

- El Coach no cambia de objetivo por una sola sesión.
- Maximiza el retorno esperado del entrenamiento, no perfecciona todas las skills.
- Cambia de objetivo lentamente; puede cambiar de interpretación rápidamente.
- Una única carrera no invalida un entrenamiento; una única victoria no
  demuestra consolidación completa.
- Jerarquía: resultados deportivos → transferencia → skills → métricas.

---

## 2. Motivación

Attribution responde *por qué*. Planner responde *cómo hoy*. Falta el juicio
estable de entrenador: foco, razón, por qué no otro, dónde validar, cuándo
revisar — sin reaccionar a cada sesión.

---

## 3. Propuesta normativa

### 3.1 Preguntas por motor

| Motor | Pregunta |
|-------|----------|
| Analysis | ¿Qué ocurrió? |
| Attribution | ¿Por qué ocurrió? |
| **Strategy** | **¿Qué debo hacer ahora?** |
| Planner | ¿Cómo entreno hoy? |
| Progress | ¿Ha funcionado el entrenamiento? |

### 3.2 Modelo

```text
DevelopmentStrategy
├── current_focus
├── focus_reason
├── why_not_other              # por qué NO otro entrenamiento
├── focus_state                # active | consolidating | validated | blocked
├── validation_context         # practice | qualifying | race
├── recommended_action         # continue | change | validate | insufficient
├── review_condition           # basada en evidencia (no N fijo de producto)
├── confidence
├── coach_plan
├── evidence_refs[] / withheld[]
└── meta (engine_version, computed_at, inputs_hash)
```

### 3.3 Decisiones de diseño cerradas

| Tema | Decisión |
|------|----------|
| Persistencia v1 | **No.** Reconstrucción on-demand |
| `review_condition` | Basada en **evidencia**, no en número fijo de sesiones como contrato |
| Skill nueva en una sesión | Solo **acumula evidencia**; no cambia el foco sola |
| Tras `validated` | Buscar siguiente limitante; si no hay → **mantener foco actual** |
| Contadores 3 Practices / 2 Races | No son ley de producto; umbrales viven como reglas versionadas en Core si se usan |

### 3.4 Conceptos prohibidos

`horizon`, `phase`, `posture`, `competition_policy`.

---

## 4. Alternativas descartadas

Estrategia en Frontend; Planner como cerebro de programa; Attribution que
también elige foco; modelo con horizon/phase/posture — todas descartadas.

---

## 5. Invariantes

1. Una pregunta: ¿Qué debo hacer ahora?
2. No `change` por una sola sesión.
3. Skill nueva ≠ cambio automático.
4. `why_not_other` siempre presente cuando hay foco activo.
5. Jerarquía deportiva → transferencia → skills → métricas.
6. Una carrera no invalida entrenamiento; una victoria no consolida sola.
7. v1 sin persistencia.
8. No ejercicios, no métricas recalculadas, no LLM.
9. Misma entrada → misma salida lógica.

---

## 6. Impacto de implementación

| Área | Impacto |
|------|---------|
| Core | `core/strategy/` on-demand |
| Backend | Adapter de lectura cuando se cablee; sin tabla nueva en v1 |
| Planner | Soft input `current_focus` / `recommended_action` |
| Frontend | Proyectar Strategy (incl. `why_not_other`) |
| Attribution / Analysis / parser | Sin cambios de contrato |

---

## 7. Orden de implementación

| Fase | Trabajo |
|------|---------|
| **0** | Docs aprobados *(hecho)* |
| **1** | Core models + hashing + tests de estabilidad / jerarquía |
| **2** | Engine: continue/change/validate/insufficient + review evidencial + why_not_other + validated→siguiente limitante |
| **3** | Backend adapter on-demand |
| **4** | Soft-wire Planner |
| **5** | Frontend consume Strategy |
| **6** | Persistencia solo si RFC futuro lo aprueba |

---

## 8. Riesgos

| Riesgo | Mitigación |
|--------|------------|
| Reintroducir N fijo como doctrina | Contrato: review evidencial |
| Cambios de foco nerviosos | Acumulación de evidencia; principio de lentitud |
| Silencio sobre “por qué no otro” | Campo `why_not_other` obligatorio con foco |
| Métricas anulando resultados | Jerarquía explícita |

---

## 9. Preguntas abiertas residuales (no bloquean el arranque)

1. Forma tipada exacta de `review_condition` en código (además del texto).
2. Momento de exponer endpoint HTTP (fase 3) vs solo uso interno inicial.
3. Copy mínimo de `coach_plan` vs proyección pura en Frontend.

Los umbrales numéricos opcionales dentro de reglas Core se definen en
implementación y tests; no reabren el contrato de producto.

---

## 10. Decisión

```text
Estado: Aprobado
Fecha: 2026-08-08
Decisión: Autorizada la implementación según §7
Notas: Modelo 3.2 — on-demand, review evidencial, why_not_other, jerarquía
```

---

## 11. Historial

| Versión | Fecha | Notas |
|---------|-------|-------|
| 003-draft | 2026-08-08 | horizon/phase/posture |
| 003-draft-3.1 | 2026-08-08 | Modelo focus_* |
| 003-approved-3.2 | 2026-08-08 | Cierre: evidencia, on-demand, acumulacion, validated, jerarquía, why_not_other |

---

*Fin — RFC 003.*
