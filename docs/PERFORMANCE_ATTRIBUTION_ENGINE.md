# DDP — Performance Attribution Engine

> Documento de arquitectura del motor de atribución de rendimiento.
> Complementa `ARCHITECTURE.md`, `ANALYSIS_ENGINE.md`, `INTERPRETATION_ENGINE.md`
> y la filosofía del Coach multidimensional (DDP 2.8).
>
> Ante conflicto con un documento congelado del Core, el documento congelado
> tiene prioridad y este diseño se corrige.

---

## Estado

| Aspecto | Situación |
|---------|-----------|
| Versión | 3.0 (`ddp-attribution-1.0.0`) |
| Estado | Implementado en Core + API + consumo Dashboard |
| Capa objetivo | `core/attribution` → `GET /drivers/{id}/attribution` → Dashboard / Coach |
| Relación | No sustituye Analysis ni Interpretation; los **atribuye** |
| Salida canónica | `PerformanceAttribution` (alias documental: AttributionResult) |

---

## 1. Objetivo

El **Performance Attribution Engine** responde a una pregunta distinta a medir
o diagnosticar una sesión:

> **¿Qué explica el resultado observado del piloto, con qué evidencia y con
> qué confianza?**

No resume al piloto con una sola etiqueta. Separa causas en dimensiones
independientes y solo afirma lo que puede sostener con hechos ya medidos o
persistidos.

```
Hechos (sesión, análisis, skills, progreso, resultados)
    → AttributionResult
    → Coach / Dashboard (narrativa, nunca al revés)
```

---

## 2. Responsabilidades del motor

### 2.1 Dentro de alcance

| Responsabilidad | Descripción |
|-----------------|-------------|
| Atribuir resultado a causas | Relacionar un outcome (carrera, clasificación, práctica) con factores candidatos |
| Separar dimensiones | Deportiva, técnica, transferencia y progreso global son lecturas distintas |
| Respetar el tipo de sesión | Practice = aprender; Qualifying = extraer ritmo; Race = competir |
| Exigir evidencia | Toda afirmación lleva refs y un nivel de confianza |
| Declinar sin evidencia | Emitir `insufficient_evidence` en lugar de inventar |
| Prioridad narrativa deportiva | En Race, el resultado competitivo manda; la telemetría lo explica, no lo contradice |
| Explicar margen restante | Una victoria no implica skills consolidadas; una skill abierta no implica retroceso |

### 2.2 Fuera de alcance

| No es responsabilidad | Motivo |
|-----------------------|--------|
| Medir telemetría / deltas / ideal | Analysis Engine |
| Diagnosticar curva prioritaria de una sesión aislada | Interpretation Engine |
| Recalcular scores EMA del perfil | Training / Profile engine |
| Generar ejercicios o planes | Training planner |
| Redactar UI o layout | Frontend |
| Usar modelos generativos opacos | Rompen trazabilidad determinista |

### 2.3 Fronteras

```
Analysis Engine        → mide (qué pasó en números)
Interpretation Engine  → diagnostica sesión (qué problema priorizar hoy)
Attribution Engine     → explica evolución y causalidad plausible
Coach / Dashboard      → narra al piloto sin contradecir atribuciones
```

El Attribution Engine **consume** salidas de Analysis, Interpretation, Skills,
Progress y resultados de sesión. No vuelve a leer `.ibt`.

---

## 3. Entradas y salidas

### 3.1 Entradas

Todas las entradas son hechos ya existentes. Ninguna se inventa en el motor.

| Entrada | Origen | Uso |
|---------|--------|-----|
| `session_context` | `session_type` clasificado | Practice / Qualifying / Race |
| `session_result` | `finish_position`, `start_position`, `incidents`, flags | Evolución y resultado deportivo |
| `session_analysis` | Analysis Engine | Ritmo, consistencia, errores, findings |
| `skill_assessments` | Driver Coach / skills | Habilidades de la sesión |
| `profile_evolution` | Dual timeline (`development` / `competitive`) | Tendencias técnicas y competitivas |
| `training_progress` | Progress Engine | Efecto del assignment activo |
| `transfer_notes` / series duales | Persistencia de evolución | Puente práctica → competición |
| `chronology` | `session.started_at` | Orden real; nunca orden de importación |

**Regla de mezcla:** Practice no entra en tendencias deportivas. Race/Qualifying
no invalidan progreso técnico de Practice.

### 3.2 Salida canónica: `PerformanceAttribution`

Implementación: `core/attribution/models.py` → `PerformanceAttribution`.
API: `GET /drivers/{driver_id}/attribution`.

```text
PerformanceAttribution
├── subject / session_kind / outcome_summary
├── reasoning                # 4 preguntas: qué / por qué / significado / ahora
├── dimensions
│   ├── sporting             # ¿compite mejor?
│   ├── technical            # ¿mejoran habilidades?
│   ├── transfer             # ¿aparece lo entrenado en competición?
│   └── global               # interpretación (no media)
├── root_causes[] / positive_factors[] / negative_factors[]
├── training_effect          # escala no binaria (+ label / confidence)
├── competition_effect       # = dimensions.sporting
├── transfer_assessment      # = dimensions.transfer
├── next_focus / coach_story / causal_chain
├── confidence / confidence_label
├── attributions[]           # claims con evidence_refs + status
├── withheld[]               # lo que se negó a concluir y por qué
├── recent_context
└── meta (engine_version, computed_at, inputs_hash)
```

### 3.3 Escalas de producto (salidas interpretadas)

**Efectividad del entrenamiento** (nunca Sí/No):

| Valor | Significado |
|-------|-------------|
| `very_effective` | Objetivo de entrenamiento alcanzado con evidencia clara |
| `effective` | Mejora observable y alineada con competición o métrica |
| `partially_effective` | Contribuye, pero la habilidad no está consolidada |
| `insufficient_evidence` | No hay base para juzgar |
| `ineffective` | Evidencia de que el foco no produjo el cambio esperado |

**Dimensiones:** cada una tiene `level`, `label`, `explanation`, `confidence`.
El progreso global **no** es el promedio aritmético de las otras tres.

---

## 4. Qué explica y qué no

### 4.1 Sí explica

| Pregunta | Ejemplo de afirmación válida |
|----------|------------------------------|
| ¿Qué ha ocurrido? | “Has conseguido la victoria.” / “Terminaste P10.” |
| ¿Por qué es plausible? | “Menor pérdida en frenada + menos incidentes respecto a la ventana previa.” |
| ¿Hay transferencia? | “El foco de throttle en Practice coincide con menos pérdidas en Race.” |
| ¿Qué sigue abierto? | “La consistencia mejora, pero aún no está consolidada.” |
| ¿Qué no se puede afirmar? | “Sin evidencia suficiente para vincular el DNF al entrenamiento.” |

### 4.2 No explica (prohibido)

| Conclusión prohibida | Por qué |
|----------------------|---------|
| “Retroceso” ante una progresión deportiva clara (P15→P1) | Confunde dimensión técnica con deportiva |
| “El entrenamiento no funcionó” solo porque una skill sigue abierta | Una skill crítica puede coexistir con mejora competitiva |
| Media única piloto = f(skills) | Borra el contexto de sesión y el resultado |
| Causa única sin alternativas | El rendimiento es multicausal |
| Intención del piloto / estado mental | No hay evidencia instrumental |
| Setup, tráfico o azar no medidos | Si no está en hechos, no se atribuye |
| Contradicción del resultado de Race | La telemetría explica el resultado; no lo anula |

### 4.3 Principio Race-first

Cuando `session_kind = race` y el resultado es claramente positivo:

1. La primera afirmación es el **resultado deportivo**.
2. Después: factores que **explican** ese resultado.
3. Después: margen técnico restante.
4. Al final: próximo entrenamiento.

Nunca al revés.

---

## 5. Cómo evita conclusiones sin evidencia

### 5.1 Contrato de evidencia

Una `attribution` solo puede tener `status = asserted` si cumple **todas**:

1. Existe al menos un `evidence_ref` resoluble a un hecho persistido o medido.
2. El hecho pertenece al **timeline correcto** (development vs competitive).
3. El tamaño de muestra supera el mínimo de la regla aplicada.
4. `confidence >= umbral_assert` (ver §6).
5. No contradice un outcome deportivo de mayor prioridad narrativa sin
   etiquetarse explícitamente como “margen técnico coexistente”.

Si falla cualquiera → `tentative` o `withheld`, nunca un titular negativo fuerte.

### 5.2 Gatekeepers (reglas duras)

| Situación | Comportamiento obligatorio |
|-----------|----------------------------|
| Menos de 2 puntos en la ventana | `insufficient_evidence` en esa dimensión |
| Race sin `finish_position` | No inventar podium/victoria; usar fallback competitivo solo como `tentative` |
| Progress `competitive_context` | No marcar entrenamiento como `ineffective` |
| Sporting `muy_positiva` / `positiva` | Prohibido emitir `ineffective` o “Retroceso” global |
| Skill estancada + resultados al alza | Explicar coexistencia; no mezclar en una sola etiqueta |
| Solo Practice disponible | No afirmar transferencia a carrera |
| Hallazgo de una sola vuelta anómala | No atribuir tendencia multi-sesión |

### 5.3 `withheld[]` explícito

El motor debe listar lo que **rehusó** concluir:

```text
withheld:
  - claim: "El entrenamiento causó la victoria"
    reason: "correlación temporal insuficiente; n_races_after_focus < 2"
  - claim: "Retroceso técnico global"
    reason: "sporting_level=muy_positiva tiene prioridad; skill abierta ≠ retroceso"
```

Esto evita el silencio engañoso: el piloto ve que hubo límite de evidencia,
no que “todo va mal”.

### 5.4 Determinismo

Misma `inputs_hash` → mismo `AttributionResult`.
Sin LLM en el camino crítico. La narrativa del Dashboard/Coach es una
proyección de este resultado, no su fuente.

---

## 6. Cómo calcula el nivel de confianza

### 6.1 Definición

`confidence ∈ [0.0, 1.0]` expresa **qué tan sostenible es la afirmación**,
no qué tan bueno es el piloto.

```
confidence = clamp01(
    0.35 * sample_factor
  + 0.25 * signal_strength
  + 0.20 * consistency_factor
  + 0.10 * recency_factor
  + 0.10 * independence_factor
)
```

| Factor | Qué mide | Alto cuando… |
|--------|----------|--------------|
| `sample_factor` | Tamaño de ventana | Hay ≥ N sesiones del timeline correcto |
| `signal_strength` | Magnitud del cambio / claridad del outcome | Delta o posición claramente direccional |
| `consistency_factor` | Estabilidad del patrón | Misma dirección en la mayoría de puntos |
| `recency_factor` | Vigencia temporal | Los hechos clave son los más recientes |
| `independence_factor` | Diversidad de evidencias | Coinciden ≥ 2 fuentes (resultado + skill + progress) |

### 6.2 Factores por tipo de afirmación

| Tipo de claim | Mínimo de muestra | Fuentes preferidas |
|---------------|-------------------|--------------------|
| Resultado de una Race | 1 sesión con posición | `finish_position`, flags |
| Tendencia deportiva | ≥ 3 Race/Qual con posición | Serie de posiciones + incidentes |
| Tendencia técnica | ≥ 3 Practice en evolution window | `window_trend`, stagnant_sessions |
| Transferencia | ≥ 1 foco de entrenamiento + ≥ 1 competitiva posterior | Dual timeline + progress |
| Efectividad de entrenamiento | Assignment + sesión de evaluación válida | Progress outcome + métrica |

### 6.3 Umbrales de emisión

| `confidence` | `status` | Uso en producto |
|--------------|----------|-----------------|
| ≥ 0.75 | `asserted` | Titular de dimensión / Coach |
| 0.50 – 0.74 | `tentative` | Matiz (“parece”, “empieza a”) |
| < 0.50 | `withheld` | “Sin evidencia suficiente” |

Alineación con skills existentes: el Driver Coach ya exige confianza mínima
para marcar strength/weakness. Attribution reutiliza esa idea: **sin confianza,
no hay juicio fuerte**.

### 6.4 Penalizaciones

| Condición | Efecto |
|-----------|--------|
| Timelines mezclados en la misma claim | `confidence = 0` (inválida) |
| Una sola fuente débil | × 0.6 |
| Contradicción entre fuentes | bajar a `tentative` o `withheld` |
| Outcome Race positivo vs skill “empeorando” | No penalizar el claim deportivo; el claim técnico va aparte |
| Datos estimados / incompletos | Techo `confidence ≤ 0.49` |

### 6.5 Confianza de dimensión vs confianza de claim

- Cada `attributions[]` tiene su propia `confidence`.
- Cada dimensión expone `confidence = max(asserted)` de sus claims
  principales, o la media ponderada si hay varios asserted.
- El progreso global usa la confianza del claim narrativo resultante,
  no el promedio ciego de dimensiones.

---

## 7. Modelo de atribución (resumen operativo)

```text
1. Clasificar sesión / ventana (Practice | Qualifying | Race)
2. Fijar outcome_summary desde hechos (posición, progreso, skills)
3. Evaluar dimensiones en aislamiento
4. Generar claims candidatos con evidence_refs
5. Calcular confidence por claim
6. Aplicar gatekeepers (Race-first, no binario, no mezcla de timelines)
7. Marcar asserted | tentative | withheld
8. Emitir AttributionResult + training_effectiveness
```

Ejemplo (P15 → P10 → P2 → P1):

| Dimensión | Lectura | Confianza típica |
|-----------|---------|------------------|
| Deportiva | Muy positiva | Alta (serie de posiciones) |
| Técnica | Puede seguir mixta / abierta | Independiente |
| Transferencia | Parcial / positiva | Media-alta si hubo foco previo |
| Global | “Progresa claramente; skills aún por consolidar” | Alta en lo deportivo |
| Entrenamiento | Parcialmente efectivo (nunca “No”) | Media |

---

## 8. Relación con el Coach y el Dashboard

| Superficie | Consume de Attribution |
|------------|------------------------|
| KPIs del Dashboard | Cuatro dimensiones + efectividad |
| Contexto reciente | Historia Practice → Qualifying → Race |
| Brief del Coach | Estructura: ocurrido → por qué → significado → qué entrenar |
| Objetivos activos | No se cierran por un mal claim técnico si la dimensión deportiva manda |

El Coach **no** inventa causalidad. Solo verbaliza `AttributionResult`.

---

## 9. Criterios de aceptación

1. Una progresión deportiva clara nunca produce titular “Retroceso”.
2. Una victoria nunca abre con “el entrenamiento no funcionó”.
3. Practice y Race no se mezclan al calcular tendencias.
4. Toda afirmación `asserted` tiene `evidence_refs` y `confidence ≥ 0.75`.
5. Sin muestra suficiente → `insufficient_evidence`, no juicio negativo.
6. Misma entrada → misma salida (`inputs_hash` estable).

---

## 10. Historial

| Versión | Fecha | Notas |
|---------|-------|-------|
| 2.8-draft | 2026-08-08 | Contrato de atribución multidimensional post Coach Intelligence |
| 3.0 | 2026-08-08 | Implementación Core + API + Dashboard; salida `PerformanceAttribution` |

---

*Fin — Performance Attribution Engine.*
