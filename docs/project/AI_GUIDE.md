# DDP — AI Guide

> Define el papel de la IA dentro de DDP.
> Documento normativo. Ante duda: la IA no manda.
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08  
**Decisión de producto:** [`DECISIONS.md`](DECISIONS.md) D-016  
**ADR:** [`ADR.md`](ADR.md) ADR-002

---

## Principio absoluto

```text
La IA es la VOZ de DDP.
Nunca su CEREBRO.
```

Los motores deterministas deciden, miden, atribuyen y eligen el foco.
La IA comunica ese conocimiento al piloto.

---

## La IA nunca

| Prohibición | Motivo |
|-------------|--------|
| **Nunca decide** | El foco y las acciones salen de Strategy / política versionada |
| **Nunca interpreta telemetría** | Analysis / Interpretation son motores de hechos y diagnóstico |
| **Nunca modifica Attribution** | Causalidad auditada; no reescribir causas en prosa libre |
| **Nunca modifica Strategy** | El programa no se negocia con un LLM |
| **Nunca genera hechos** | Sin evidencia en Facts/BD/Core → no se afirma |
| **Nunca inventa evidencias** | Confianza = trazabilidad |
| **Nunca elige el foco en la UI** | Proyección solamente |

Si una capacidad propuesta viola esta tabla → **no pertenece a DDP**.

---

## La IA únicamente

| Capacidad | Significado |
|-----------|-------------|
| **Comunica** | Traduce salidas de motor a lenguaje de piloto |
| **Explica** | Ayuda a entender el «por qué» ya calculado |
| **Enseña** | Orienta el aprendizaje sin cambiar el programa |
| **Resume** | Condensa historia y estado sin alterar hechos |
| **Conversa** | Diálogo natural anclado a evidencia |
| **Acompaña** | Presencia continua (visión: tiempo real / mentor) |

La Narrative Intelligence actual del frontend es un precursor **determinista**
de esta capa: ya humaniza sin LLM. Cualquier LLM futuro debe respetar
las mismas fronteras.

---

## Arquitectura futura

```text
Motores deterministas
        ↓
AI Communication Layer
        ↓
Conversation Engine
        ↓
Real Time Engineer
        ↓
Mentor Deportivo
```

| Capa | Rol |
|------|-----|
| **Motores deterministas** | Import, Analysis, Interpretation, Attribution, Strategy Facts, Development Strategy, Planner, Progress, Profile |
| **AI Communication Layer** | Voz: explica y narra salidas ya calculadas |
| **Conversation Engine** | Preguntas del piloto → respuestas ancladas a Facts/Strategy/Attribution |
| **Real Time Engineer** | Acompañamiento durante la sesión (tiempo real) |
| **Mentor Deportivo** | Arco a largo plazo; compañero de carrera |

Ninguna capa superior puede saltarse los motores para “decidir mejor”.

---

## Contrato con el resto del sistema

```text
Hechos + Attribution + Strategy + Planner
              │
              ▼
     AI Communication Layer   ← solo lectura de salidas
              │
              ▼
        Piloto / UI
```

- Entrada de la IA: estructuras ya producidas por el Core/Backend.
- Salida de la IA: texto / voz / diálogo.
- Efectos prohibidos: escribir Strategy, Attribution, Facts, Analysis, perfil.

---

## Relación con el Coach actual

Hoy el Coach de superficie (Dashboard briefing, Explainability, Journey)
es **proyección determinista** + Narrative Intelligence.

Eso es correcto y permanece válido aunque más adelante exista un LLM:

1. Primero los motores producen la verdad.
2. Después la capa de comunicación la dice bien.
3. Nunca al revés.

Ver: `COACH_PHILOSOPHY.md`, Fases B–D, `NARRATIVE_INTELLIGENCE.md`.

---

## Criterio de aceptación para cualquier feature de IA

Antes de implementar:

1. ¿Qué motor produce el hecho o la decisión?
2. ¿La IA solo lo comunica?
3. ¿Puede el piloto pedir el «por qué» anclado a evidencia?
4. ¿Misma evidencia → misma decisión de motor (aunque cambie el fraseo)?

Si (2) es no → rediseñar.

---

## Visión alineada

Coach en tiempo real → Ingeniero de pista → IA conversacional → Mentor deportivo.

Detalle de producto: [`ROADMAP.md`](ROADMAP.md).  
Filosofía: [`DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md).

---

*La IA representa la voz de DDP. Nunca su cerebro.*
