# DDP — Filosofía del Coach

> Documento de producto y criterio de coaching.
> Complementa `ARCHITECTURE.md`, `PERFORMANCE_ATTRIBUTION_ENGINE.md`,
> `DDP_2_DEVELOPMENT_PROGRAM.md` y `DEVELOPMENT_STRATEGY_ENGINE.md`.
>
> Ante conflicto con un documento congelado del Core, el documento congelado
> tiene prioridad y este diseño se corrige.

---

## Estado

| Aspecto | Situación |
|---------|-----------|
| Versión | 3.2 |
| Estado | **Arquitectura aprobada** — lista para implementación |
| Ámbito | Producto / voz / criterios de decisión del Coach |
| Relación | Define *cómo piensa* DDP; no define APIs ni esquemas |

---

## 1. Propósito

DDP no es un visualizador de telemetría con frases encima.

Es un **ingeniero de pista + entrenador** que ha revisado el historial del
piloto y puede explicar, con evidencia:

1. qué ha ocurrido,
2. por qué es plausible que haya ocurrido,
3. qué significa para su desarrollo,
4. qué conviene hacer a continuación.

La sensación objetivo:

> Hablar con alguien que ha visto todas las sesiones, entiende la relación
> entre entrenamiento, habilidades, competición y evolución, y no inventa
> lo que no puede demostrar.

DDP no cambia continuamente de opinión. Mantiene un objetivo mientras exista
evidencia de que sigue siendo el mayor limitante. Solo cambia cuando la
evidencia demuestra que el cuello de botella ha cambiado.

Debe comportarse como un **entrenador real**, no como un algoritmo que
reacciona a cada sesión.

---

## 2. Una pregunta por motor

Cada módulo debe responder a **una única pregunta**.

| Motor | Pregunta |
|-------|----------|
| Analysis Engine | ¿Qué ocurrió? |
| Interpretation Engine | ¿Qué problema priorizar en esta sesión? |
| Performance Attribution Engine | ¿Por qué ocurrió? |
| Development Strategy Engine | ¿Qué debo hacer ahora? |
| Training Planner | ¿Cómo entreno hoy? |
| Progress Engine | ¿Ha funcionado el entrenamiento? |
| Driver Profile | ¿Cómo evolucionan las skills en el tiempo? |
| Coach / Dashboard | ¿Cómo se lo digo al piloto sin contradecir los hechos? |

**Regla dura:** la UI y el Coach de superficie **no inventan causalidad ni
estrategia**. Solo verbalizan Attribution y Strategy.

---

## 3. Principios fundamentales

### 3.1 Estabilidad de objetivo

**EL COACH NO CAMBIA DE OBJETIVO POR UNA SOLA SESIÓN.**

Solo cambia el foco cuando exista **evidencia suficiente**.

Pensamiento canónico:

> “Mientras esta habilidad siga limitando el rendimiento,
> seguiremos trabajando en ella.”

> “Cuando deje de limitar el rendimiento,
> cambiaremos de objetivo.”

Consecuencias:

1. Una sesión mala no basta para cambiar el foco.
2. Una sesión buena no basta para dar el foco por cerrado.
3. Una skill nueva señalada en una sesión **solo acumula evidencia**; no
   cambia el foco automáticamente.
4. Sin evidencia suficiente → mantener el foco actual o declarar
   `insufficient`.

### 3.2 Objetivo del entrenamiento

**El objetivo del Coach es maximizar el retorno esperado del entrenamiento,
no perfeccionar todas las skills.**

Se entrena lo que más limita el rendimiento ahora. No se persigue cerrar el
catálogo de habilidades. Una skill “abierta” que ya no limita puede esperar.

### 3.3 Velocidad distinta: interpretación vs objetivo

**DDP cambia de objetivo lentamente, pero puede cambiar de interpretación
rápidamente.**

- Tras cada sesión puede actualizarse la lectura (Attribution, diagnóstico).
- El foco de desarrollo (`current_focus`) permanece estable hasta que la
  evidencia demuestre otro cuello de botella.

### 3.4 Una sesión no sentencia

**Una única carrera no invalida un entrenamiento, y una única victoria no
demuestra que una habilidad esté completamente consolidada.**

Race-first en narrativa ≠ cambio automático de programa tras un solo resultado.

### 3.5 Jerarquía de decisión

En caso de conflicto entre señales, siempre prevalece el nivel superior:

```text
Resultados deportivos
        ↓
Transferencia
        ↓
Skills
        ↓
Métricas
```

| Nivel | Pregunta que manda |
|-------|--------------------|
| Resultados deportivos | ¿El piloto compite mejor? |
| Transferencia | ¿Lo entrenado aparece bajo presión? |
| Skills | ¿Qué habilidad limita ahora? |
| Métricas | ¿Qué números lo sostienen? |

Una métrica o skill aislada **no** puede anular una progresión deportiva clara.
Una victoria **no** obliga a declarar todas las skills consolidadas (ver §3.4).

---

## 4. Principios operativos

### 4.1 Determinismo y evidencia

- Misma evidencia → misma conclusión.
- Toda afirmación fuerte lleva evidencia y confianza.
- Sin evidencia suficiente → decirlo. No inventar tendencias.
- Prohibido LLM en el camino crítico de juicio.

### 4.2 Race-first (prioridad narrativa deportiva)

Cuando el outcome reciente es una Race con resultado claro:

1. Primero el **resultado deportivo**.
2. Después los factores que lo **explican**.
3. Después el **margen técnico** restante.
4. Al final el **siguiente trabajo**.

La telemetría explica la carrera; no la contradice.

### 4.3 Dimensiones independientes

Nunca una sola etiqueta para todo el piloto.

| Dimensión | Pregunta |
|-----------|----------|
| Deportiva | ¿Compite mejor? |
| Técnica | ¿Mejoran las habilidades en Practice? |
| Transferencia | ¿Lo entrenado aparece en competición? |
| Global | Interpretación narrativa (no media aritmética) |

### 4.4 Separación Practice / Competición

- Practice = entrenar.
- Qualifying = confirmar ritmo.
- Race = validar bajo presión.

### 4.5 Lenguaje de pista

Prohibido como titular: “No mejoraste.”, “Retroceso.”, “Skill 62.”

Preferido: qué cambió, por qué, qué margen queda, cuál es el foco y por qué
**no** se recomienda otro entrenamiento ahora.

### 4.6 Un solo foco

Un objetivo de desarrollo activo. Sin rotación por sesión.

### 4.7 Honestidad sobre incertidumbre

> “No existe evidencia suficiente.”

---

## 5. Las cuatro preguntas de toda narrativa

1. **¿Qué cambió?**
2. **¿Por qué cambió?** → Attribution
3. **¿Qué significa?**
4. **¿Qué hacemos ahora?** → Strategy

Incluye, cuando aplique, **por qué no** otro entrenamiento.

---

## 6. Quién decide qué

| Decisión | Dueño |
|----------|--------|
| Medir | Analysis |
| Diagnóstico de sesión | Interpretation |
| Señal de skill de sesión | Training Coach |
| Ejercicio del día | Training Planner |
| ¿Funcionó el assignment? | Progress |
| Tendencias de perfil | Profile |
| ¿Por qué ocurrió? | Attribution |
| ¿Qué debo hacer ahora? (foco estable) | **Strategy** |
| Palabras / layout | Frontend |

---

## 7. Criterios de buen coaching

Bueno si:

1. Un entrenador humano reconoce la lógica.
2. Respeta la jerarquía resultados → transferencia → skills → métricas.
3. Maximiza retorno del entrenamiento, no perfección total.
4. Cambia interpretación rápido; objetivo lento.
5. Explica también por qué **no** cambia de foco.
6. No sentencia por una sola carrera ni consolida por una sola victoria.

Malo si:

1. Cambia el foco tras cada sesión.
2. Deja que una métrica anule un resultado deportivo claro.
3. Persigue cerrar todas las skills.
4. Planifica por semanas o fases artificiales.

---

## 8. Career Mode (futuro)

```text
PerformanceAttribution  →  DevelopmentStrategy  →  narrativa / hitos
```

Sin segundo cerebro.

---

## 9. Gobernanza

| Cambio | Requiere |
|--------|----------|
| Matiz de voz | Este documento |
| Decisión de foco | RFC 003 + `DEVELOPMENT_STRATEGY_ENGINE.md` |
| Afirmación causal | Attribution |
| Ejercicio / curvas | DDP 2.0 / training |

---

## 10. Historial

| Versión | Fecha | Notas |
|---------|-------|-------|
| 3.0-draft | 2026-08-08 | Post Attribution |
| 3.1-draft | 2026-08-08 | Simplificación focus_* |
| 3.2 | 2026-08-08 | Aprobada: jerarquía, retorno, ritmo interpretación/objetivo, regla 1 carrera/1 victoria |

---

*Fin — Coach Philosophy.*
