# DDP — Development Strategy Engine

> Documento de arquitectura del motor de estrategia de desarrollo.
> Complementa `COACH_PHILOSOPHY.md`, `PERFORMANCE_ATTRIBUTION_ENGINE.md`,
> `ARCHITECTURE.md` y `DDP_2_DEVELOPMENT_PROGRAM.md`.
>
> Ante conflicto con un documento congelado del Core, el documento congelado
> tiene prioridad y este diseño se corrige.
>
> Diseño aprobado. Ver RFC 003. Implementación pendiente.

---

## Estado

| Aspecto | Situación |
|---------|-----------|
| Versión | 3.2 |
| Estado | **Arquitectura aprobada** — lista para implementación |
| Capa objetivo | Core (`core/strategy/` propuesto) |
| Persistencia v1 | **Ninguna** — se reconstruye on-demand |
| Pregunta única | **¿Qué debo hacer ahora?** |

---

## 1. Objetivo

> **¿Qué debo hacer ahora?**

Concreto: foco actual, por qué, estado, dónde se valida, cuándo revisar,
qué acción recomendar — y **por qué no** otro entrenamiento.

Pensamiento de entrenador:

> “Mientras esta habilidad siga limitando el rendimiento, seguiremos
> trabajando en ella. Cuando deje de limitar, cambiaremos de objetivo.”

```text
Hechos + PerformanceAttribution
        │
        ▼
DevelopmentStrategy   (on-demand, sin persistir en v1)
        │
        ├── Training Planner
        ├── Coach / Dashboard
        └── Future Career Mode
```

Principios normativos: `COACH_PHILOSOPHY.md` §3.

---

## 2. Principios del motor

1. **No cambia de objetivo por una sola sesión.**
2. **Maximiza el retorno esperado del entrenamiento**, no perfecciona todas
   las skills.
3. **Cambia de objetivo lentamente; la interpretación puede cambiar rápido**
   (Attribution se refresca; el foco permanece).
4. **Una única carrera no invalida un entrenamiento; una única victoria no
   demuestra consolidación completa.**
5. **Jerarquía de decisión** (conflicto → gana el nivel superior):

```text
Resultados deportivos
        ↓
Transferencia
        ↓
Skills
        ↓
Métricas
```

6. Una skill nueva en una sesión **solo acumula evidencia**; no cambia el foco
   automáticamente.
7. Tras `validated`, buscar el siguiente limitante; si no hay uno claro,
   **mantener el foco actual**.

---

## 3. Responsabilidades

### 3.1 Dentro de alcance

| Responsabilidad | Descripción |
|-----------------|-------------|
| `current_focus` | Habilidad que más limita el rendimiento ahora |
| `focus_reason` | Por qué ese foco (evidencia) |
| `why_not_other` | Por qué **no** se recomienda otro entrenamiento ahora |
| `focus_state` | `active` / `consolidating` / `validated` / `blocked` |
| `validation_context` | Dónde comprobar si funciona (Practice / Qual / Race) |
| `recommended_action` | `continue` / `change` / `validate` / `insufficient` |
| `review_condition` | Condición de revisión **basada en evidencia**, no en N fijo |
| `coach_plan` | Plan corto y estable |
| `confidence` | Sostenibilidad del juicio |

### 3.2 Fuera de alcance

| No hace | Dueño |
|---------|--------|
| Medir telemetría | Analysis |
| Diagnosticar sesión | Interpretation |
| Explicar causas | Attribution |
| Generar ejercicios | Planner |
| Persistir Strategy en v1 | — (on-demand) |
| Planificar semanas | Fuera de producto |
| LLM | Prohibido |

### 3.3 Conceptos eliminados

`horizon`, `phase`, `posture`, `competition_policy` — no reintroducir.

---

## 4. Fronteras

```text
Analysis        → ¿Qué ocurrió?
Interpretation  → ¿Qué problema priorizar en esta sesión?
Attribution     → ¿Por qué ocurrió?          (puede cambiar rápido)
Strategy        → ¿Qué debo hacer ahora?     (cambia lento)
Planner         → ¿Cómo entreno hoy?
Progress        → ¿Ha funcionado el entrenamiento?
```

### 4.1 Con Attribution

| Regla | Detalle |
|-------|---------|
| Dirección | Strategy consume Attribution; nunca al revés |
| Velocidad | Attribution se recalcula por sesión; Strategy mantiene foco salvo evidencia |
| Jerarquía | Strategy aplica la jerarquía deportiva → transferencia → skills → métricas |
| Race-first | Un mal día en Race no tumba el programa; una victoria no cierra el foco sola |

---

## 5. Entradas

| Entrada | Uso |
|---------|-----|
| `performance_attribution` | Limitantes, transfer, training_effect, withheld |
| `driver_profile` / evolution dual | ¿Sigue limitando? ¿Hay otro cuello de botella? |
| `active_assignment` | Foco en curso |
| `training_history` | Evidencia acumulada de focos previos (no calendario) |
| `progress_snapshot` | ¿Funcionó el entrenamiento reciente? |
| `session_context` + chronology | Orden real de evidencia |

Señal de skill de una sesión nueva: **evidencia acumulada**, no trigger de cambio.

---

## 6. Salida: `DevelopmentStrategy`

```text
DevelopmentStrategy
├── current_focus
│   ├── skill_id
│   └── label
├── focus_reason                 # por qué este foco
├── why_not_other                # por qué NO otro entrenamiento ahora
├── focus_state                  # active | consolidating | validated | blocked
├── validation_context           # practice | qualifying | race
├── recommended_action           # continue | change | validate | insufficient
├── review_condition             # condición basada en evidencia (texto + forma tipada)
├── confidence
├── coach_plan
├── evidence_refs[]
├── withheld[]
└── meta
    ├── engine_version
    ├── computed_at
    └── inputs_hash
```

**v1:** se calcula on-demand; no se persiste.

---

## 7. Conceptos

### 7.1 `current_focus`

Objetivo activo mientras limite el rendimiento (según jerarquía §2).

### 7.2 `focus_reason`

Por qué existe el foco, anclado a evidencia.

Ejemplo: *“Actualmente esta habilidad es la que más limita el rendimiento
competitivo.”*

### 7.3 `why_not_other`

Explicación explícita de por qué **no** se recomienda otro entrenamiento.

Ejemplos:

- “Otra skill aparece en la última Practice, pero una sola sesión no basta
  para cambiar el objetivo.”
- “Hay margen en acelerador, pero la entrada sigue explicando mejor las
  pérdidas bajo presión.”
- “Los resultados deportivos ya mejoran con el foco actual; cambiar ahora
  reduciría el retorno esperado del entrenamiento.”

Sin este campo, el Coach parece arbitrario.

### 7.4 `focus_state`

| Valor | Significado |
|-------|-------------|
| `active` | Limitante actual; se entrena |
| `consolidating` | Mejora en curso; aún no cerrado |
| `validated` | Dejó de limitar en el contexto de validación |
| `blocked` | No hay base para avanzar el juicio |

**Tras `validated`:** buscar el siguiente limitante claro.
Si no existe → **mantener el foco actual** (`recommended_action = continue`
o equivalente estable; no inventar un cambio vacío).

### 7.5 `validation_context`

¿Dónde comprobamos si el entrenamiento funciona?

| Valor | Rol |
|-------|-----|
| `practice` | Entrenar / comprobar aprendizaje |
| `qualifying` | Confirmar |
| `race` | Validar bajo presión |

### 7.6 `recommended_action`

| Valor | Significado |
|-------|-------------|
| `continue` | Seguir con el mismo foco |
| `change` | Cambiar de foco (evidencia suficiente + siguiente limitante claro) |
| `validate` | Mantener foco; comprobar en `validation_context` |
| `insufficient` | No hay base para decidir |

### 7.7 `review_condition` (basada en evidencia)

**No** es “cada N sesiones” como regla fija de producto.

Es una condición evaluable sobre evidencia, por ejemplo:

- el foco deja de aparecer como limitante en competición de forma sostenida
- el entrenamiento cumple objetivo y la transferencia acompaña
- emerge un nuevo cuello de botella con evidencia multi-sesión
- Attribution + jerarquía señalan otro factor dominante

Los umbrales concretos viven como reglas versionadas en Core (no como
“siempre 3 Practices / 2 Races” en el contrato de producto). La documentación
exige el *tipo* de condición (evidencial), no un contador rígido universal.

Mientras no se cumpla → no `change` por inercia.

---

## 8. Decisiones que SÍ / NO toma

**Sí:** foco, razón, por qué no otro, estado, validación, acción, condición
de revisión, plan, confianza.

**No:** medir, reinterpretar causas, ejercicios, persistir en v1, planificar
semanas, cambiar foco por una sesión, cerrar skills por una victoria,
invalidar entrenamiento por una carrera.

---

## 9. Modelo operativo

```text
1. Leer Attribution + foco actual + progress/profile
2. Aplicar jerarquía: deportivo → transferencia → skills → métricas
3. ¿El foco actual sigue limitando?
4. Si una sesión señala otra skill → acumular evidencia; no change
5. Resolver focus_reason + why_not_other + focus_state
6. Resolver validation_context + recommended_action
7. Si focus_state = validated → buscar siguiente limitante
      · existe → change (si review_condition satisfecha)
      · no existe → mantener foco actual
8. Emitir review_condition (evidencial) + coach_plan + DevelopmentStrategy
```

---

## 10. Training Planner

Soft input: preferir `current_focus` cuando `recommended_action != insufficient`.

Cambio de skill en Planner solo alineado con `recommended_action = change`.

---

## 11. Coach / Dashboard

| Campo | Uso en UI |
|-------|-----------|
| `current_focus` + `focus_reason` | Objetivo actual |
| `why_not_other` | Por qué no otro entrenamiento |
| `recommended_action` + `coach_plan` | Qué hacer ahora |
| `validation_context` | Dónde lo comprobamos |
| `review_condition` | Cuándo lo revisaremos |

---

## 12. Criterios de aceptación

1. Pregunta única: ¿Qué debo hacer ahora?
2. Sin horizon/phase/posture/competition_policy.
3. v1 on-demand, sin persistencia.
4. Una sesión no provoca `change`.
5. Skill nueva acumula evidencia solamente.
6. Tras `validated`, siguiente limitante o mantener foco.
7. `review_condition` evidencial, no N fijo de producto.
8. Existe `why_not_other`.
9. Jerarquía deportiva → transferencia → skills → métricas.
10. Misma entrada → misma salida (`inputs_hash`).

---

## 13. Ubicación (implementación)

```text
core/strategy/
```

Backend adapter / API: fases posteriores explícitas; v1 puede exponerse cuando
se acuerde el cableado, siempre calculando on-demand.

---

## 14. Historial

| Versión | Fecha | Notas |
|---------|-------|-------|
| 3.0-draft | 2026-08-08 | Contrato inicial |
| 3.1-draft | 2026-08-08 | Modelo focus_* |
| 3.2 | 2026-08-08 | Aprobada: review evidencial, on-demand, why_not_other, jerarquía, validated→siguiente limitante |

---

*Fin — Development Strategy Engine.*
