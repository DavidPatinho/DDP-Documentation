# DDP 4.0 — Driver Experience · Fase A

**Dashboard Experience — Centro de mando del piloto**

Fecha: 2026-08-08  
Alcance: solo frontend / proyección UI. Arquitectura de motores **congelada**.

---

## Objetivo de la fase

Convertir el Dashboard de panel de datos en el centro de mando del piloto, de modo que en ~30 segundos responda:

1. ¿Dónde estoy?
2. ¿Estoy mejorando?
3. ¿Qué estoy entrenando?
4. ¿Qué debo hacer ahora?

Pregunta de pantalla actualizada: **«¿Qué necesito saber hoy?»**

---

## Archivos modificados

| Archivo | Cambio |
|---------|--------|
| `frontend/src/pages/Dashboard.tsx` | Reestructura completa de jerarquía visual (mando) |
| `frontend/src/pages/dashboard/dashboardExperience.ts` | **Nuevo** — proyección UI (objetivo, headline, evolución sentida, progreso, carrera) |
| `frontend/src/pages/dashboard/loadDashboard.ts` | Ensambla campos Fase A sin tocar motores |
| `frontend/src/theme/layout.ts` | `screenQuestions.dashboard` → «¿Qué necesito saber hoy?» |
| `frontend/src/theme/navigation.ts` | Subtítulo → «Centro de mando del piloto» |
| `docs/DRIVER_EXPERIENCE_PHASE_A.md` | Este informe |

### No modificado (congelado)

Analysis Engine · Performance Attribution Engine · Strategy Facts Provider · Development Strategy Engine · Training Planner (arquitectura) · Parser · SessionAnalysis.

---

## Decisiones tomadas

1. **Proyección, no motor.** Toda la experiencia nueva vive en `dashboardExperience.ts` + UI. Strategy / Attribution siguen siendo la única fuente de foco y causalidad.
2. **Jerarquía de mando (arriba → abajo):**
   - Estado + headline del día
   - Objetivo actual (con criterios de éxito) + Qué hacer ahora (`FocusCallout`)
   - Evolución sentida (frases) + máx. 3 skills relevantes
   - Última carrera + Último progreso
   - Coach (voz continua; briefing narrativo profundo = Fase B)
3. **Objetivo en lenguaje de entrenador.** No «Entrena throttle» → «Consolidar el control del acelerador.» Criterios: deja de ser limitante · transferencia · validación (`review_condition`).
4. **Menos ruido.** Se eliminan del primer viewport: 4 KPI MetricCards, grid de hasta 6 skills, bloque «Contexto reciente» duplicado, lista «Objetivos activos» genérica, campos técnicos del aside (why-not-other / efectividad / validar en).
5. **Evolución como frases.** Las dimensiones Attribution se muestran como oraciones de entrenador, no como tarjetas KPI.
6. **Skills de mando.** Solo limitante + progreso (+ 1 consolidada opcional), máximo 3.

---

## Impacto sobre la experiencia del piloto

| Antes | Ahora |
|-------|--------|
| Panel denso de KPIs y skills | Centro de mando con una lectura de 30 s |
| «Entrenamiento recomendado» técnico | «Qué debo hacer ahora» protagonista |
| Objetivos como lista de skills | Un objetivo claro + cómo saber que está conseguido |
| Contexto repetido (hero + sección) | Última carrera y último progreso, una vez cada uno |
| Pregunta: cómo evoluciono / qué entrenar | Pregunta: qué necesito saber **hoy** |

El piloto abre DDP y ve de inmediato estado, objetivo, evolución y próximo paso — sin buscar.

---

## Capturas de pantalla

Pendientes de captura en runtime local (abrir `/` con backend + sesiones incorporadas).

Checklist visual sugerido:

1. Hero con nombre + headline
2. Objetivo actual con 3 criterios
3. FocusCallout «Qué debo hacer ahora»
4. Bloque Evolución (frases)
5. Última carrera / Último progreso

---

## Checklist UX (Fase A)

- [x] La primera pantalla responde «¿Qué necesito saber hoy?»
- [x] Estado actual del piloto visible sin scroll
- [x] Objetivo actual en lenguaje natural (no «Entrena X»)
- [x] Criterios de éxito del objetivo visibles
- [x] Próximo entrenamiento / acción protagonista
- [x] Evolución como narrativa (no solo KPIs)
- [x] Última carrera contextualizada
- [x] Último progreso visible
- [x] Sin duplicar Practice/Quali/Race en dos sitios
- [x] Sin inventar foco ni causalidad en UI
- [x] Motores congelados intactos
- [ ] Capturas runtime (cuando haya entorno con datos)
- [ ] Validación con piloto real (30 s)

---

## Criterio de éxito (fase)

Si un piloto puede abrir el Dashboard 30 segundos y responder:

- dónde está
- qué ha mejorado
- qué le limita
- qué debe entrenar hoy

→ **Fase A cumplida a nivel de producto.** Pendiente validación humana con datos reales.

---

## Siguiente fase

**Fase B — Coach Experience:** convertir el bloque Coach en un briefing natural continuo (un discurso, no frases sueltas ni métricas repetidas), alineado con la voz del mando.
