# DDP 4.0 — Driver Experience · Fase D

**Driver Journey — Memoria deportiva del piloto**

Fecha: 2026-08-08  
Alcance: capa de proyección en presentación. Sin motores nuevos. Sin APIs nuevas.

**Fases A (Dashboard), B (Coach) y C (Explainability): congeladas.**  
No se modificaron Dashboard, Coach briefing ni Explainability.

---

## Objetivo de la fase

Que un piloto que lleve meses usando DDP abra **Trayectoria** y entienda en menos de dos minutos:

- cómo empezó  
- qué habilidades desarrolló  
- cuándo cambiaron sus objetivos  
- cuándo llegaron sus primeros resultados  
- qué hitos marcaron su evolución  
- dónde está ahora  
- cuál es el siguiente gran paso  

Pregunta de pantalla: **«¿Cómo he llegado hasta aquí?»**

No es un Timeline de sesiones. No es una lista de IBT. Es una biografía deportiva.

---

## Archivos creados / modificados

| Archivo | Cambio |
|---------|--------|
| `frontend/src/pages/driver/driverJourney.ts` | **Nuevo** — proyección de hitos y etapas (sin cálculos de motor) |
| `frontend/src/pages/driver/loadDriverJourney.ts` | **Nuevo** — ensambla Facts / Strategy / Attribution / Evolution / History / Sessions |
| `frontend/src/pages/Driver.tsx` | Sustituye stub vacío por pantalla Driver Journey |
| `frontend/src/types/strategyFacts.ts` | Tipado ampliado del payload Facts ya expuesto por la API |
| `frontend/src/theme/layout.ts` | `screenQuestions.driverJourney` |
| `frontend/src/theme/navigation.ts` | Nav «Trayectoria» + subtítulo memoria deportiva |
| `docs/UI_GUIDELINES.md` | Pregunta de pantalla Trayectoria |
| `docs/DRIVER_EXPERIENCE_PHASE_D.md` | Este informe |

### No modificado (congelado)

- Dashboard Experience (`Dashboard.tsx`, `dashboardExperience.ts`)
- Coach Experience (`coachBriefing.ts`)
- Explainability Experience (`explainability.ts`, `WhyExplanationSheet.tsx`)
- Analysis Engine · Performance Attribution Engine · Strategy Facts Provider · Development Strategy Engine · Training Planner
- Pipeline / backend (la API Strategy ya devolvía `facts.as_dict()` completo)

---

## Decisiones de diseño

1. **Proyección, no motor.** Toda la inteligencia ya existía. Journey solo selecciona eventos que cambian el desarrollo y los narra.
2. **Ruta `/driver`.** Reutiliza el stub «Piloto» → ahora «Trayectoria». No se tocó el Dashboard.
3. **Hitos, no sesiones.** Solo eventos con evidencia (primera telemetría, primer entrenamiento, cambio de foco, mejora validada, podio, victoria, skill validada, objetivo cerrado, nuevo objetivo, dónde estás ahora).
4. **Etapas de desarrollo, no calendario.** Capítulos: Inicio → Aprendizaje → Consolidación → Competición → Nuevo desafío. Solo aparecen si tienen hitos.
5. **Cada hito responde cinco preguntas.** Qué ocurrió · Por qué importó · Qué cambió · Qué aprendiste · Después.
6. **Logros visuales.** Victoria, podio, skill consolidada, objetivo conseguido, mayor mejora, cambio importante — con badge y énfasis verde.
7. **Enlace al Coach.** Cada hito puede abrir la sesión asociada (`/sessions/:id`) para recordar qué se decidió entrenar.
8. **No inventar.** Sin evidencia → sin hito. Sin posiciones de llegada → sin podio/victoria. Sin progress positivo → sin mejora consolidada.
9. **Sin IDs ni métricas crudas.** Hashes, `skill_id` técnicos y deltas numéricos no se muestran.

### Fuentes consumidas

```
Timeline (cronología GET /sessions + Facts)
  → Analysis (contexto de sesión)
  → Attribution (cierre «dónde estás ahora»)
  → Strategy Facts (validaciones, foco, competición)
  → Development Strategy (foco actual, plan, estado)
  → Planner (historial de assignments / objetivos)
```

---

## Ejemplos con datos reales

Extraídos del piloto **David** en la base local (`database/ddp.db`, 7 sesiones, driver_id=3):

### Capítulo 1 — Inicio
**Así empezó todo**  
Registraste tu primera telemetría en Mount Panorama Circuit.

### Capítulo 2 — Aprendizaje
**Cuando empezaste a entrenar con propósito**  
Tu primer entrenamiento dirigido: Control del acelerador — básico.  
**Cuando cambió el objetivo por primera vez**  
El foco pasó de Control del acelerador a Entrada.

### Capítulo 3 — Consolidación
**Cuando Entrada dio el salto** *(logro: Mayor mejora registrada)*  
Evidencia de Evolution (delta/tendencia). Sin `validated_sessions` positivos → no se inventó «mejora consolidada» por progress.

### Capítulo 4 — Nuevo desafío
**Dónde estás ahora**  
Narrativa Attribution: hay sesiones competitivas (2 qualifying) pero sin posiciones finales suficientes.  
Foco Strategy actual: Entrada en curva — avanzado.  
**Siguiente gran paso** → `coach_plan` de Strategy.

### Lo que no aparece (correcto)
- Primer podio / primera victoria → no hay carreras con `finish_position`.  
- Skill validada → `validated_in_competition` no es true.  
- Objetivo completado → assignments en `superseded`, no `completed`.

---

## Checklist UX

- [ ] La pantalla responde solo a «¿Cómo he llegado hasta aquí?»
- [ ] No parece una lista/tabla de sesiones ni de ficheros IBT
- [ ] Los capítulos se leen como evolución (Inicio → … → Nuevo desafío)
- [ ] Cada hito tiene qué / por qué / qué cambió / qué aprendió / después
- [ ] Los logros destacan visualmente (badge + acento)
- [ ] No se muestran hashes, IDs ni métricas crudas
- [ ] Los enlaces «Ver qué explicó el Coach» abren la sesión asociada
- [ ] Sin evidencia suficiente → empty state honesto, sin hitos ficticios
- [ ] El cierre indica dónde está ahora y el siguiente gran paso
- [ ] Dashboard / Coach / Explainability intactos

---

## Capturas

Capturadas en runtime local (`http://127.0.0.1:5173/driver`, piloto David):

1. Hero — pregunta «¿Cómo he llegado hasta aquí?» + arco narrativo + Capítulo Inicio  
2. Capítulos Aprendizaje (primer entrenamiento + cambio de foco)  
3. Logro «Mayor mejora registrada» (Entrada) + Capítulo Nuevo desafío  
4. Cierre FocusCallout «Siguiente gran paso» → Strategy `coach_plan`  

Empty state: verificable sin sesiones (mensaje honesto, sin hitos ficticios).

---

## Impacto

| Antes | Ahora |
|-------|--------|
| `/driver` stub vacío | Memoria deportiva del piloto |
| Histórico solo como lista de sesiones | Historia por etapas e hitos |
| Sin lectura de «cómo llegué aquí» | Biografía en &lt; 2 minutos |

---

## Siguiente fase

No continuar automáticamente. Fase D entregada.
