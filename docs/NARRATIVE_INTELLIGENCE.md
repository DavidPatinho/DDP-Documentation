# DDP 4.0 — Narrative Intelligence

Fecha: 2026-08-08  
Alcance: capa de prosa piloto (frontend). Sin motores nuevos. Sin IA.

**No modificado:** Analysis · Attribution · Strategy · Planner · Strategy Facts (motores).

---

## Objetivo

Que el Coach escriba como un ingeniero de pista que recuerda al piloto, no como un sistema que describe estados.

Orden obligatorio de toda historia:

**HECHO → CONSECUENCIA → INTERPRETACIÓN → PRÓXIMO PASO**

---

## Archivos modificados

| Archivo | Cambio |
|---------|--------|
| `frontend/src/lib/narrative.ts` | **Nuevo** — fechas, progresiones, humanización, filtro de frases vacías |
| `frontend/src/pages/dashboard/coachBriefing.ts` | Briefing reescrito con hechos / fechas / comparaciones |
| `frontend/src/pages/dashboard/dashboardExperience.ts` | Headline, objetivo, carrera y progreso con hechos |
| `frontend/src/pages/dashboard/evolutionCoach.ts` | Conclusiones de skill + dimensiones con contraste concreto |
| `frontend/src/pages/dashboard/explainability.ts` | Evidencia y criterios sin jerga Practice/Race / limitante |
| `frontend/src/pages/dashboard/loadDashboard.ts` | Pasa hechos al briefing; humaniza foco; sustituye prosa vacía de Attribution |
| `frontend/src/pages/driver/driverJourney.ts` | Cierre y limpieza alineados con la misma voz |
| `docs/NARRATIVE_INTELLIGENCE.md` | Este informe |

---

## Decisiones tomadas

1. **No tocar motores.** Si Attribution/Strategy emiten prosa vacía, la capa narrativa la descarta y reconstruye con hechos ya cargados (fechas, circuito, curva, foco previo, ventana de evolución).
2. **Conservar números útiles.** Se dejan de borrar `ms`, segundos y `%` en el clean del briefing; solo se elimina jerga interna (`focus_state`, hashes, snake_case…).
3. **Fechas reales.** Formato `7 agosto 2026` / `hace 12 días` cuando el ISO existe.
4. **Comparar, no describir.** Cambio de foco = “Antes X; desde fecha pasó a Y”. Skills = ventana + tendencia. Carreras = progresión `P15 → P10 → P2 → P1` si hay posiciones.
5. **Filtrar frases vacías de sistema** (`Hay sesiones competitivas…`, `Actualmente…`, `Seguir trabajando…`, jerarquías internas).
6. **Variedad en el cierre.** El próximo paso parte del `coach_plan` / `next_action`, reescrito; no se inventa causalidad.

---

## Antes / después (piloto David, datos reales)

### Briefing

**Antes**
> Hay sesiones competitivas, pero todavía no hay posiciones finales suficientes…  
> Actualmente Entrada en curva — avanzado es la que más limita…  
> Seguir trabajando Entrada…

**Después**
> El 7 agosto 2026 en Mount Panorama Circuit entrenaste con foco en Entrada en curva — avanzado; la zona que más pesó fue Forrest's Elbow.  
> Antes el foco era Control del acelerador; desde 7 agosto 2026 pasó a Entrada en curva — avanzado.  
> En entrenamiento, Entrada en curva — avanzado ya responde al trabajo de las últimas 6 sesiones.  
> Próximo paso: trabajar Entrada… La señal de cierre será que Forrest's Elbow deje de concentrar la pérdida principal.

### Frases genéricas eliminadas / filtradas

- “Hay sesiones competitivas…”
- “El progreso es/global es estable…”
- “Actualmente … sigue siendo el principal limitante”
- “Seguir trabajando…”
- “Ya aparece mejoría en Practice / Race”
- “Deje de aparecer como principal limitante”
- Jerarquía interna `deportiva → transferencia → skills → métricas`
- Aperturas “En este momento / Hoy por hoy”

### Reglas narrativas añadidas

1. Toda afirmación preferirá un ancla: fecha, circuito, curva, posición, ventana de N sesiones o contraste Antes/Ahora.
2. Orden fijo HECHO → CONSECUENCIA → INTERPRETACIÓN → PRÓXIMO PASO.
3. Si no hay hecho → no se afirma (frase vacía → fallback o silencio).
4. Lenguaje interno nunca llega al piloto.
5. El Coach cierra siempre con un próximo paso accionable.
6. Misma voz en Dashboard, Explainability y Journey (`humanizePilotText`).

---

## Checklist UX

- [x] El briefing se lee como historia, no como estados
- [x] Aparecen fechas / circuito / curva cuando existen
- [x] Hay contraste Antes → Ahora en cambio de foco
- [x] No se muestran `focus_state`, hashes ni jerga interna
- [x] No queda “Seguir trabajando…” / “Hay sesiones competitivas…” en el briefing
- [x] El cierre indica cómo se dará por cerrado el objetivo
- [x] Motores Analysis / Attribution / Strategy / Planner / Facts intactos
- [x] Sin datos inventados cuando falta evidencia

---

No continuar automáticamente con otra fase.
