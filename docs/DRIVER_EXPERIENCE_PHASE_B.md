# DDP 4.0 — Driver Experience · Fase B

**Coach Experience — Briefing profesional continuo**

Fecha: 2026-08-08  
Alcance: capa de presentación / narrativa. Sin motores nuevos. Sin APIs nuevas.

**Fase A (Dashboard Experience): congelada.**  
No se alteró la jerarquía del centro de mando (estado, objetivo, evolución, carrera/progreso).

---

## Objetivo de la fase

El Coach deja de listar frases técnicas y pasa a mantener una **conversación** con el piloto: un único briefing que interpreta, recuerda y enlaza con el plan.

Estructura fija (no negociable):

1. ¿Qué ha ocurrido?
2. ¿Por qué ha ocurrido?
3. ¿Qué significa para tu evolución?
4. ¿Qué debes hacer ahora?

---

## Archivos modificados

| Archivo | Cambio |
|---------|--------|
| `frontend/src/pages/dashboard/coachBriefing.ts` | **Nuevo** — composer del briefing (4 actos, memoria, contexto Practice/Quali/Race) |
| `frontend/src/pages/dashboard/loadDashboard.ts` | Sustituye el ensamblado suelto Strategy/Attribution por `composeCoachBriefing` |
| `frontend/src/pages/dashboard/composeCoachBrief.ts` | Fachada deprecada → apunta al nuevo composer |
| `frontend/src/pages/Dashboard.tsx` | Solo bloque Briefing: discurso continuo + contexto (Entrenamiento / Rendimiento / Competición) |
| `docs/DRIVER_EXPERIENCE_PHASE_B.md` | Este informe |

### No modificado

- Dashboard Experience (zonas Fase A)
- `dashboardExperience.ts`
- Performance Attribution Engine
- Strategy Facts Provider
- Development Strategy Engine
- Training Planner / Parser / SessionAnalysis

---

## Decisiones tomadas

1. **Proyección, no motor.** El briefing se compone en frontend a partir de Attribution (`reasoning`, factores, `coach_story`, transfer) + Strategy (`coach_plan`, `focus_reason`, `review_condition`, acción) + evolución proyectada ya cargada.
2. **Un discurso, cuatro párrafos.** Orden fijo. Deduplicación por solapamiento. Limpieza de métricas crudas (`%`, `0.18 s`, `skill_id`).
3. **Logros primero.** En Race/Quali se reconoce victoria, podio, pole o posición antes de explicar el siguiente paso.
4. **Memoria temporal.** Anclas del tipo «Durante las últimas N sesiones…», «Hace unas tres semanas…», «Desde que comenzaste…», «Tras consolidar…» — solo si hay fechas / window / estado que lo sostengan.
5. **Contexto por tipo de sesión.**
   - Practice → Entrenamiento
   - Qualifying → Rendimiento
   - Race → Competición
6. **Acto 4 completo.** No «Entrena X»: continuidad (`Seguimos…` / `Ahora podemos pasar a…` / `Validar en…`) + plan Strategy + criterio de revisión.
7. **Sin evidencia → honestidad.** Mensajes explícitos de «todavía es pronto» / «necesitamos más sesiones». `evidenceRefs` se conservan en el view model para Fase D.
8. **UI mínima.** El centro de mando Fase A no se reordena; solo el bloque inferior pasa a llamarse Briefing y se lee como prosa continua (sin pie «Revisión:» suelto).

---

## Ejemplos de briefing

### Practice · Entrenamiento

> El trabajo sobre control del acelerador ya muestra progreso claro en entrenamiento.  
> El progreso viene del foco mantenido sobre control del acelerador en las tandas de entrenamiento.  
> Durante las últimas 6 sesiones de entrenamiento, el trabajo sobre control del acelerador está dando resultado en entrenamiento.  
> Seguimos trabajando sobre control del acelerador como único foco. Mantendremos este objetivo hasta confirmar que también está consolidado bajo presión.

### Qualifying · Rendimiento

> Has cerrado la clasificación en P2: buen ritmo competitivo en una vuelta.  
> El margen en clasificación sigue ligado sobre todo a entrada.  
> Esta clasificación confirma el nivel actual; el siguiente salto pasa por consolidar entrada bajo presión.  
> Seguimos trabajando sobre entrada como único foco. Mantendremos este objetivo hasta confirmar que la mejora se sostiene en clasificación.

### Race · Competición

> Has conseguido la victoria tras varias carreras de progresión constante.  
> El trabajo realizado sobre entrada empieza a reflejarse en competición.  
> Aunque control del acelerador sigue siendo la habilidad con mayor margen de mejora, ya no está limitando tus resultados como antes.  
> Seguimos trabajando sobre control del acelerador como único foco. Mantendremos este objetivo hasta confirmar que también está consolidado bajo presión.

*(Los textos reales dependen de Strategy/Attribution del piloto; estos ejemplos ilustran el tono y la estructura.)*

---

## Impacto sobre la experiencia

| Antes | Ahora |
|-------|--------|
| Campos Strategy concatenados + story suelta | Un briefing de 4 actos |
| «Validar en: Race» técnico | Continuidad en lenguaje de ingeniero |
| Sin memoria temporal | Anclas de evolución cuando hay evidencia |
| Misma voz para Practice/Quali/Race | Narrativa diferenciada por contexto |
| Pie «Revisión:» desconectado | Criterio de éxito dentro del acto 4 |

---

## Checklist UX

- [x] Un único briefing (discurso), no tarjetas inconexas
- [x] Orden fijo: ocurrido → por qué → significado → ahora
- [x] Logros reconocidos antes del siguiente paso
- [x] Memoria / continuidad cuando hay evidencia
- [x] Narrativa distinta Practice / Qualifying / Race
- [x] Sin métricas crudas ni `skill_id` en el texto
- [x] Explica qué hacer, qué espera y cómo sabrá que funciona
- [x] Dice «todavía es pronto» si falta evidencia
- [x] No inventa foco ni causalidad
- [x] Fase A / motores intactos
- [ ] Capturas runtime con datos reales
- [ ] Lectura piloto &lt; 1 minuto (validación humana)

---

## Capturas

Pendientes con entorno local + sesiones incorporadas. Verificar `data-testid="coach-briefing-discourse"`.

---

## Criterio de éxito (fase)

El piloto lee el Briefing en menos de un minuto y siente que el Coach:

- recuerda su evolución  
- entiende qué ocurre y por qué  
- tiene un plan claro  
- le guía para mejorar  

→ **Fase B cumplida a nivel de producto** (pendiente validación con datos reales).

---

## No continuar automáticamente

Siguiente fase prevista (solo bajo petición): **Fase C — Timeline Experience**.
