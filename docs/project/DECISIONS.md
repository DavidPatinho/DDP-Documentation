# DDP — Decisiones de producto

> Registro permanente de decisiones de producto.
> No confundir con [`ADR.md`](ADR.md) (decisiones técnicas de arquitectura).
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08

---

## D-001 — DDP desarrolla pilotos, no analiza vueltas

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Misión: desarrollo de pilotos |
| **Descripción** | El producto es la evolución del piloto a largo plazo. La telemetría es materia prima, no el destino. |
| **Motivación** | Evitar que DDP derive en visor de gráficos o panel de métricas. |
| **Impacto** | Toda funcionalidad se filtra por «¿ayuda al piloto a ser más completo, rápido e inteligente?» |
| **Estado** | Vigente |
| **Documentos** | `DDP_PHILOSOPHY.md` |
| **RFC** | — |

---

## D-002 — Confianza mediante evidencia y explicación

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Sin explicación no hay confianza |
| **Descripción** | Toda recomendación, mantenimiento o cambio de foco debe poder justificarse. El piloto puede preguntar «¿por qué?» |
| **Motivación** | La confianza es un pilar; sin ella el Coach se ignora. |
| **Impacto** | Explainability (Fase C); evidencia en Strategy/Attribution; rechazo de consejos opacos. |
| **Estado** | Vigente |
| **Documentos** | `DDP_PHILOSOPHY.md`, `DRIVER_EXPERIENCE_PHASE_C.md` |
| **RFC** | — |

---

## D-003 — El Coach no cambia de objetivo por una sola sesión

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Estabilidad de foco de entrenamiento |
| **Descripción** | El foco solo cambia con evidencia suficiente. Una sesión mala/buena no basta. Una skill nueva solo acumula evidencia. |
| **Motivación** | Comportarse como entrenador real, no como algoritmo reactivo. |
| **Impacto** | Development Strategy; jerarquía deportiva → transferencia → skills → métricas. |
| **Estado** | Vigente |
| **Documentos** | `COACH_PHILOSOPHY.md`, `DEVELOPMENT_STRATEGY_ENGINE.md` |
| **RFC** | RFC 003 |

---

## D-004 — Maximizar retorno del entrenamiento, no perfeccionar todas las skills

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Entrenar el limitante actual |
| **Descripción** | Se entrena lo que más limita el rendimiento ahora. Una skill abierta que ya no limita puede esperar. |
| **Motivación** | Evitar programas que intentan «cerrar el catálogo» en lugar de ganar tiempo en pista. |
| **Impacto** | `why_not_other` obligatorio; Strategy elige foco único. |
| **Estado** | Vigente |
| **Documentos** | `COACH_PHILOSOPHY.md`, RFC 003 |
| **RFC** | RFC 003 |

---

## D-005 — Race-first en narrativa, no en cambio de programa

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Prioridad narrativa deportiva |
| **Descripción** | En Race, el resultado competitivo manda en la historia. Una victoria no consolida todas las skills ni cambia el foco sola. |
| **Motivación** | El piloto merece reconocimiento deportivo sin sabotear el programa. |
| **Impacto** | Attribution dimensions; Coach abre con resultado en Race/Quali. |
| **Estado** | Vigente |
| **Documentos** | `PERFORMANCE_ATTRIBUTION_ENGINE.md`, `COACH_PHILOSOPHY.md`, Fase B |
| **RFC** | — |

---

## D-006 — Dashboard como centro de mando, no panel de KPIs

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase A |
| **Descripción** | El Dashboard responde en ~30 s: dónde estoy, si mejoro, qué entreno, qué hacer ahora. Pregunta: «¿Qué necesito saber hoy?» |
| **Motivación** | Menos ruido, más decisión. |
| **Impacto** | Eliminación de KPI clutter; objetivo + FocusCallout protagonistas. |
| **Estado** | Vigente · experiencia congelada |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_A.md` |
| **RFC** | — |

---

## D-007 — Coach como briefing continuo de cuatro actos

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase B |
| **Descripción** | Un discurso: qué ocurrió → por qué → significado → qué hacer ahora. Contexto Practice/Quali/Race. |
| **Motivación** | Dejar de listar frases técnicas; hablar como ingeniero de pista. |
| **Impacto** | `coachBriefing.ts`; estructura narrativa fija. |
| **Estado** | Vigente · experiencia congelada |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_B.md`, `NARRATIVE_INTELLIGENCE.md` |
| **RFC** | — |

---

## D-008 — Explainability integrada, sin pantalla nueva

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase C |
| **Descripción** | Sheet lateral «¿Por qué?» con evidencia, confianza, why_not_other y cuándo cambia el foco. Sin IDs/hashes al piloto. |
| **Motivación** | Confianza sin saturar el centro de mando. |
| **Impacto** | `WhyExplanationSheet`; Facts + Strategy proyectados. |
| **Estado** | Vigente · experiencia congelada |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_C.md` |
| **RFC** | — |

---

## D-009 — Trayectoria como biografía deportiva, no lista de IBT

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase D |
| **Descripción** | `/driver` narra hitos y etapas (Inicio → … → Nuevo desafío). Sin evidencia → sin hito. |
| **Motivación** | El piloto debe entender «cómo he llegado hasta aquí» en &lt; 2 minutos. |
| **Impacto** | Journey proyecta; no inventa podios ni validaciones. |
| **Estado** | Vigente |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_D.md` |
| **RFC** | — |

---

## D-010 — Strategy sustituye weaknesses como fuente de foco

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Foco único desde Development Strategy |
| **Descripción** | El Dashboard y el Coach toman el foco de Strategy usable, no de `weaknesses[0]` ni listas de debilidades. |
| **Motivación** | Una sola fuente de verdad de programa; evitar contradicciones. |
| **Impacto** | Priority shift solo si Strategy `action === "change"`. |
| **Estado** | Vigente |
| **Documentos** | `AUDIT_REPORT_RCA1.md`, `DEVELOPMENT_STRATEGY_ENGINE.md` |
| **RFC** | RFC 003 |

---

## D-011 — Lectura HTTP no muta el perfil del piloto

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | GET ≠ escritura (RCA-1) |
| **Descripción** | Abrir Dashboard/Sesiones no reincorpora sesiones. Incorporar es un acto explícito (POST). |
| **Motivación** | «Quitar del perfil» debe persistir; los GET no pueden deshacer decisiones del piloto. |
| **Impacto** | `persist=False` por defecto; `POST …/profile/incorporate`. |
| **Estado** | Vigente |
| **Documentos** | `AUDIT_REPORT_RCA1.md` |
| **RFC** | — |

---

## D-012 — Una historia por piloto activo

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Multi-piloto con piloto activo de producto |
| **Descripción** | Dashboard y Journey filtran por piloto activo seleccionable en Settings. |
| **Motivación** | Evitar biografías mezcladas entre pilotos. |
| **Impacto** | API `GET/PUT /drivers/active`; loaders filtrados. |
| **Estado** | Vigente |
| **Documentos** | `AUDIT_REPORT_RCA1.md` |
| **RFC** | — |

---

## D-013 — Informes fuera del nav en Release Candidate

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Retirar stub de Informes del nav |
| **Descripción** | La entrada Informes generaba expectativa falsa; se retira del nav RC. |
| **Motivación** | Honestidad de producto: no prometer pantallas vacías. |
| **Impacto** | Nav RC sin Informes; reports weekly/career siguen como stubs de Core. |
| **Estado** | Vigente (RC) |
| **Documentos** | `AUDIT_REPORT_RCA1.md` |
| **RFC** | — |

---

## D-014 — DDP 2.0 simplificado: skill → curvas → ejercicio

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Mapa circuito–skill–curvas (v1.1) |
| **Descripción** | Sustituye complejidad de programa semanal/career tables. Planner ancla ejercicios a curvas del circuito. |
| **Motivación** | Entregar valor de entrenamiento sin un segundo producto. |
| **Impacto** | `circuit_skill_map` + YAML; RFC 002. |
| **Estado** | Vigente |
| **Documentos** | `DDP_2_DEVELOPMENT_PROGRAM.md` |
| **RFC** | RFC 002 |

---

## D-015 — Project Memory System como memoria oficial

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | PMS v1 forma parte de la arquitectura |
| **Descripción** | La memoria del proyecto vive en `docs/project/`. Ninguna fase se cierra sin actualizar el PMS. Las conversaciones de IA no son la fuente de verdad. |
| **Motivación** | Supervivencia del conocimiento a 6 meses / 2 años sin depender de chats. |
| **Impacto** | Checklist obligatorio de cierre; orden de lectura de arranque. |
| **Estado** | Vigente |
| **Documentos** | `docs/project/*`, `ARCHITECTURE.md`, `README.md` |
| **RFC** | — |

---

## D-016 — La IA es la voz de DDP, nunca su cerebro

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Papel de la IA en el producto |
| **Descripción** | La IA comunica, explica, enseña, resume, conversa y acompaña. Nunca decide, interpreta telemetría, genera hechos ni modifica Attribution/Strategy. |
| **Motivación** | Preservar determinismo, trazabilidad y confianza. |
| **Impacto** | Arquitectura futura: motores → AI Communication Layer → Conversation → Real Time → Mentor. |
| **Estado** | Vigente |
| **Documentos** | [`AI_GUIDE.md`](AI_GUIDE.md) |
| **RFC** | — |

---

## D-017 — PMS pasa a ser infraestructura oficial (PMI)

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Project Memory Infrastructure (PMI v1) |
| **Descripción** | El mantenimiento del PMS deja de depender solo de la disciplina humana. Existe infraestructura en `tools/pms/`, especificación `PMS_SPEC.md` y procedimiento oficial `/close-phase`. |
| **Motivación** | Que la memoria del proyecto se mantenga con el mismo rigor que los tests. |
| **Impacto** | Cierre de fase = procedimiento PMI; validación automática; scripts append-only para DECISIONS/ADR. |
| **Estado** | Vigente |
| **Documentos** | `PMS_SPEC.md`, `tools/README.md`, `ARCHITECTURE.md` |
| **RFC** | RFC — Project Memory Infrastructure (PMI v1) |

---

## D-018 — AI Context System como puerta oficial al PMS

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | AICS v1 — entrada diferenciada humanos / IA |
| **Descripción** | Humanos empiezan por `BOOTSTRAP.md`. Las IA empiezan por `AI_BOOTSTRAP.md` y profundizan en el PMS solo cuando hace falta. El AICS no sustituye al PMS. |
| **Motivación** | Reconstruir el contexto del proyecto en &lt; 1 minuto sin depender de chats previos. |
| **Impacto** | `/close-phase` mantiene `AI_BOOTSTRAP` si cambia el contexto; validate_pms exige entradas AICS. |
| **Estado** | Vigente |
| **Documentos** | `BOOTSTRAP.md`, `AI_BOOTSTRAP.md`, `PMS_SPEC.md` |
| **RFC** | RFC — AI Context System (AICS v1) |

---

## D-019 — Configuración global única y prompt ChatGPT generado

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | AICS v2 — PROJECT_CONFIGURATION + ChatGPT Startup Prompt |
| **Descripción** | Toda URL/repo compartido vive solo en `PROJECT_CONFIGURATION.md`. El prompt oficial de ChatGPT se genera automáticamente y vive en `AI_BOOTSTRAP.md`. Existe AI INITIALIZATION PROTOCOL obligatorio. |
| **Motivación** | Incorporar IA sin chats previos; evitar URLs duplicadas y prompts obsoletos. |
| **Impacto** | `/close-phase` regenera el prompt; validate_pms exige sync con la configuración. |
| **Estado** | Vigente |
| **Documentos** | `PROJECT_CONFIGURATION.md`, `AI_BOOTSTRAP.md`, `PMS_SPEC.md` |
| **RFC** | RFC — AI Context System (AICS v2) |

---

## D-020 — Integración GitHub de documentación para ChatGPT

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | AICS v2.1 — repo público de docs como fuente para ChatGPT |
| **Descripción** | El prompt oficial inyecta Official Documentation Repository, branch y paths desde PROJECT_CONFIGURATION. ChatGPT reconstruye el contexto desde ese repo GitHub. |
| **Motivación** | Que el prompt funcione de verdad con ChatGPT sin editar prompts a mano. |
| **Impacto** | Cambio de URL = solo PROJECT_CONFIGURATION + regenerar prompt; validate_pms prohíbe URLs duplicadas. |
| **Estado** | Vigente |
| **Documentos** | `PROJECT_CONFIGURATION.md`, `AI_BOOTSTRAP.md`, `PMS_SPEC.md` |
| **RFC** | RFC — AICS v2.1 GitHub Documentation Integration |

---

*Añadir nuevas decisiones al final. No borrar las vigentes; marcar estado si se superseden.*
