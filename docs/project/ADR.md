# DDP — Architecture Decision Records

> Decisiones técnicas importantes.
> No confundir con [`DECISIONS.md`](DECISIONS.md) (producto).
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08

---

## ADR-001 — Core independiente sin HTTP, UI ni SQLite

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | La lógica de dominio no debe morir con el framework de turno. |
| **Decisión** | El Core solo depende de Pandas, NumPy y stdlib. Entrada/salida: estructuras Python puras. Sin efectos (BD, red, UI). |
| **Consecuencias** | Backend es adaptador. Tests de motor sin servidor. Frontend intercambiable. |

---

## ADR-002 — La IA nunca decide

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Modelos generativos opacos rompen trazabilidad y confianza. |
| **Decisión** | Las decisiones de foco, causalidad y hechos las toman motores deterministas. La IA (cuando exista) solo comunica. |
| **Consecuencias** | Misma evidencia → misma conclusión. Ver [`AI_GUIDE.md`](AI_GUIDE.md). |

---

## ADR-003 — La UI nunca interpreta

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Si el Dashboard recalcula causalidad o dimensiones, contradice al Core y al Coach. |
| **Decisión** | Frontend proyecta Attribution y Strategy. Si Attribution falla → estado «insuficiente», no fallback local. |
| **Consecuencias** | Corregido en RCA-1 (A3). Experience layers son proyección. |

---

## ADR-004 — Strategy es la única fuente del foco

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Usar `weaknesses[0]` u otras listas genera focos inconsistentes. |
| **Decisión** | `current_focus` y acción de programa salen de Development Strategy. Priority shift solo si `recommended_action === "change"`. |
| **Consecuencias** | Dashboard/Coach alineados. Planner no redefine el foco. |

---

## ADR-005 — La cronología utiliza `started_at`

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | El orden de importación no es el orden deportivo real. |
| **Decisión** | Toda cronología de perfil/evolución usa `session.started_at`. Import out-of-order exige rebuild. |
| **Consecuencias** | EMA y skills coherentes con la historia real del piloto. |

---

## ADR-006 — Analysis genera hechos de medición

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Mezclar medición y opinión hace imposible auditar números. |
| **Decisión** | Analysis Engine mide (ideal, consistencia, findings). No coachea. No escribe narrativa de programa. |
| **Consecuencias** | Frontera clara con Interpretation y Attribution. Doc `ANALYSIS_ENGINE.md` v1.0 congelada. |

---

## ADR-007 — Attribution interpreta causalidad

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Hace falta explicar *por qué* cambió el rendimiento sin re-medir telemetría. |
| **Decisión** | Performance Attribution consume hechos ya medidos/persistidos y emite `PerformanceAttribution` con dimensiones, confianza y `insufficient_evidence` cuando no hay base. |
| **Consecuencias** | Coach/Dashboard narran Attribution; no la inventan. |

---

## ADR-008 — Planner nunca cambia Strategy

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Si el planner elige el programa, hay dos cerebros. |
| **Decisión** | Planner responde «¿cómo entreno hoy?» a partir del foco de Strategy. No redefine `current_focus`. |
| **Consecuencias** | Cadena Strategy → Planner → Assignment. |

---

## ADR-009 — Interpretation diagnostica sesión, no programa

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Confundir diagnóstico de tanda con foco de desarrollo. |
| **Decisión** | Interpretation Engine prioriza problema de **esta sesión**. El programa multi-sesión es Strategy. |
| **Consecuencias** | Velocidad distinta: interpretación puede cambiar rápido; foco lento. Doc v1.0 congelada. |

---

## ADR-010 — Strategy v1 on-demand sin persistencia

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Persistir Strategy prematuramente acopla esquema a un juicio aún en evolución. |
| **Decisión** | Development Strategy se reconstruye on-demand (RFC 003). Sin tabla Strategy en v1. |
| **Consecuencias** | Siempre refleja Facts/Attribution actuales; sin drift de filas stale. |

---

## ADR-011 — Contrato Core con `inputs_hash` + `engine_version`

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Regeneración y determinismo deben ser verificables. |
| **Decisión** | Motores de import/análisis/interpretación (y Strategy) exponen versión + hash de entradas. Misma evidencia → mismo resultado. |
| **Consecuencias** | Reinterpretar/reanalizar es seguro; regresiones posibles. |

---

## ADR-012 — Persistencia sustituye por política, no acumula duplicados opacos

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Reanálisis / reimportación no deben dejar estados ambiguos. |
| **Decisión** | Analysis: política sustituir (D8). Interpretation: regeneración en el sitio (D5). Import: dedup por `content_hash`. |
| **Consecuencias** | Un resultado coherente por sesión/motor; ver docs de cada motor. |

---

## ADR-013 — Lecturas HTTP no tienen efectos secundarios de perfil

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | GET con `persist=True` reescribía el perfil al abrir pantallas (RCA-1 C1). |
| **Decisión** | Endpoints de lectura de training/skills/progress no persisten por defecto. Escritura solo vía POST incorporate / rebuild / remove. |
| **Consecuencias** | «Quitar del perfil» es estable. |

---

## ADR-014 — Delete de sesión reconstruye cronología antes del hard delete

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Borrar una sesión intermedia contaminaba EMA/skills posteriores. |
| **Decisión** | `delete_session` = remove + rebuild (con `exclude_session_id`) + hard delete. Rebuild vacío limpia career e índices. |
| **Consecuencias** | Historia coherente; deudas M4/M5 documentadas en KNOWN_ISSUES. |

---

## ADR-015 — Project Memory System es parte de la arquitectura

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | El conocimiento vivía en chats y en la cabeza de las personas. |
| **Decisión** | `docs/project/` es memoria obligatoria. Cierre de fase exige checklist PMS. |
| **Consecuencias** | Toda fase futura actualiza STATE, HANDOVER, SESSION_NOTES, etc. |

---

## ADR-016 — Conceptos prohibidos en Strategy

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Modelos con `horizon` / `phase` / `posture` / `competition_policy` sobrecargaban el juicio. |
| **Decisión** | Esos conceptos no se reintroducen en Development Strategy. |
| **Consecuencias** | Modelo simple: focus, reason, why_not_other, state, action, review_condition. |

---

## ADR-017 — Project Memory Infrastructure en tools/pms

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | El PMS documental solo se mantenía por disciplina; era fácil cerrar fases sin actualizar la memoria. |
| **Decisión** | Crear PMI en `tools/pms/` (fuera del runtime). `validate_pms` + `close_phase` son el procedimiento oficial `/close-phase`. DECISIONS/ADR solo crecen (append). `publish_docs` queda reservado para un futuro repo público de documentación. |
| **Consecuencias** | Infra versionada con el repo privado; no toca Core/Frontend/Backend funcional; cierre de fase gateado por validación PMS. |

---

## ADR-018 — AICS: AI_BOOTSTRAP como entrada canónica para IA

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Una IA nueva tenía que leer muchos documentos o depender de historial de chat para entender DDP. |
| **Decisión** | Crear AICS dentro del PMS: `AI_BOOTSTRAP.md` (IA) y `BOOTSTRAP.md` (humanos). PMS sigue siendo la única fuente de verdad. `/close-phase` invoca `update_ai_bootstrap` cuando el contexto general cambia. |
| **Consecuencias** | Reconstrucción &lt; 1 min; menos drift por conversaciones; mantenimiento AICS acoplado al cierre de fase. |

---

## ADR-019 — Una sola fuente de configuración para el AICS

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | URLs y prompts hardcodeados en varios docs se desincronizan y obligan a editar a mano. |
| **Decisión** | `PROJECT_CONFIGURATION.md` es la única fuente de `Official Documentation Repository`. PMI (`update_ai_bootstrap`) inyecta ese valor en el bloque `ChatGPT Startup Prompt` de `AI_BOOTSTRAP.md`. Ningún otro documento escribe la URL manualmente. |
| **Consecuencias** | Cambio de repo = un solo edit + regenerar prompt; validate_pms falla si el prompt está desfasado. |

---

## ADR-020 — Prompt ChatGPT inyecta config GitHub completa

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Un prompt con URL fija o incompleta deja de funcionar al mover el repo de documentación. |
| **Decisión** | El generador lee Project Name, Version, Official Documentation Repository, Branch, Docs Path, AI Bootstrap Path y Bootstrap Path desde PROJECT_CONFIGURATION.md e inyecta todos en el bloque ChatGPT Startup Prompt. validate_pms prohíbe duplicar la URL fuera de config + bloque generado. |
| **Consecuencias** | Cambio de repo = un edit en PROJECT_CONFIGURATION + `--refresh-prompt`; ChatGPT siempre recibe la config vigente. |

---

*Nuevos ADR al final. Si se supersede uno, marcar Estado = Superada y enlazar el nuevo.*
