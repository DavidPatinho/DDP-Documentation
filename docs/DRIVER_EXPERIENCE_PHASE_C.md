# DDP 4.0 — Driver Experience · Fase C

**Explainability Experience — Confianza mediante evidencia**

Fecha: 2026-08-08  
Alcance: capa de explicación en presentación. Sin motores nuevos. Sin APIs nuevas.

**Fases A (Dashboard) y B (Coach): congeladas.**  
No se reordenó el centro de mando ni se reescribió el briefing.

---

## Objetivo de la fase

Que el piloto pueda pulsar **«¿Por qué?»** y entender, con evidencia real:

- por qué entrena lo que entrena  
- por qué no otra habilidad  
- qué evidencia usa DDP  
- con qué confianza  
- cuándo cambiará el foco  

Sin confiar “porque sí”.

---

## Archivos modificados / nuevos

| Archivo | Cambio |
|---------|--------|
| `frontend/src/pages/dashboard/explainability.ts` | **Nuevo** — proyección de explicación (sin cálculos de foco) |
| `frontend/src/components/explainability/WhyExplanationSheet.tsx` | **Nuevo** — Sheet lateral «Ver explicación» |
| `frontend/src/types/strategyFacts.ts` | **Nuevo** — tipado ligero de Facts para UI |
| `frontend/src/types/strategy.ts` | `facts` tipado con `StrategyFactsPayload` |
| `frontend/src/pages/dashboard/loadDashboard.ts` | Captura Facts + ensambla `explanation` |
| `frontend/src/pages/Dashboard.tsx` | Botones «¿Por qué?» / «Ver explicación» + Sheet (sin romper jerarquía) |
| `docs/DRIVER_EXPERIENCE_PHASE_C.md` | Este informe |

### No modificado

- `dashboardExperience.ts` (Fase A)
- `coachBriefing.ts` (Fase B)
- Analysis / Attribution / Strategy Facts / Development Strategy / Planner (motores)

---

## Decisiones tomadas

1. **Sin pantalla nueva.** Sheet lateral integrado en el Dashboard.
2. **Sin cálculos nuevos.** Solo organiza Strategy (`focus_reason`, `why_not_other`, `review_condition`, `confidence_label`, `evidence_refs`, `withheld`) + Facts (`evidence_summary`, `competition_summary`) + Attribution (dimensiones / transferencia) + evolución ya proyectada.
3. **StrategyTrace.** El Trace es interno al Core (`as_dict` no lo expone; solo `as_debug_dict`). No se abrió API nueva ni se rompió el contrato. La UI usa los campos piloto que el Trace produce. Los `evidence_refs` se humanizan; hashes / `rule_id` / IDs opacos **nunca** se muestran.
4. **Evidencia comprensible.** Ej.: «8 sesiones analizadas», «3 carreras», «mejora confirmada en entrenamientos», «transferencia parcial», «objetivo aún no validado».
5. **Confianza + motivo.** Etiquetas del motor + frase breve (sesiones / insuficiencia / withheld).
6. **Cuándo cambia el foco.** Criterios explícitos + `review_condition` de Strategy.
7. **Sin inventar.** Si falta Strategy → lo dice. Withheld se lista como «Lo que aún no afirmamos».

---

## Ejemplos reales (estructura)

### ¿Por qué entreno control del acelerador?

1. *(focus_reason de Strategy)*  
2. Durante las últimas 6 sesiones ha seguido siendo el mayor limitante.  
3. Ya aparece mejoría en Practice.  
4. Todavía no está consolidado en Race.

### ¿Por qué no otra habilidad?

- *(why_not_other de Strategy, p. ej. retorno mayor de la entrada / limitante actual)*  
- Cambiaremos cuando control del acelerador deje de ser el principal limitante.

### Evidencia

- 8 sesiones analizadas  
- 3 carreras  
- Mejora confirmada en entrenamientos  
- Transferencia parcial a competición  
- Objetivo aún no validado  

### Confianza

**Alta** — Porque la tendencia se mantiene desde hace 7 sesiones.

### Cuándo cambiará el foco

Seguiremos trabajando este objetivo hasta que:

- [ ] *(review_condition)*  
- [ ] Deje de aparecer como principal limitante  
- [ ] La mejora se confirme en carrera  
- [ ] Exista otro aspecto con mayor retorno esperado  

---

## Impacto

| Antes | Ahora |
|-------|--------|
| La recomendación se aceptaba por autoridad | Se puede abrir la evidencia en un clic |
| `why_not_other` oculto en el flujo | Visible en la explicación |
| Confianza solo en JSON | Confianza + motivo en lenguaje piloto |
| Sin «cuándo cambia» | Criterios claros de salida del foco |

---

## Checklist UX

- [x] Botón «¿Por qué?» / «Ver explicación» donde importa (objetivo, foco, briefing)
- [x] Sin pantalla nueva; no rompe el flujo del Dashboard
- [x] Por qué este foco (cadena)
- [x] Por qué no otra habilidad
- [x] Evidencia humana (sin IDs / hashes / reglas)
- [x] Nivel de confianza + motivo
- [x] Cuándo cambiará el foco
- [x] Honestidad si falta evidencia / withheld
- [x] Motores y Fases A/B intactos
- [ ] Capturas runtime con datos reales
- [ ] Validación piloto: «tiene sentido»

---

## Capturas

Pendientes con entorno local. Test ids:

- `why-explanation-trigger`
- `why-explanation-sheet`

---

## Criterio de éxito

Al pulsar «¿Por qué me recomiendas esto?» el piloto debe pensar: **«Tiene sentido.»**

No: «No sé de dónde ha salido.»

---

## No continuar automáticamente

Siguiente fase solo bajo petición (p. ej. Timeline Experience si se retoma el plan original).
