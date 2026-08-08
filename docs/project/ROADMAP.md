# DDP — Roadmap de producto

> Evolución del **producto**, no tareas técnicas.
> Para aplazamientos técnicos de esquema/pipeline ver también
> [`docs/ROADMAP.md`](../ROADMAP.md) (apuntes de ingeniería).
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08

---

## Lo conseguido

### Fundación (v0.1–v0.4)
- Infraestructura local (SQLite, persistencia, verificación)
- Importación de telemetrías iRacing (`.ibt`)
- Medición objetiva de sesión (Analysis)
- Diagnóstico de sesión (Interpretation / Ingeniero de sesión)

### Programa de desarrollo (DDP 2–3)
- Skills, planner, progress y perfil dual
- Mapa circuito → skill → curvas → ejercicio
- Performance Attribution (por qué cambió el rendimiento)
- Development Strategy (qué debo hacer ahora)
- Strategy Facts como base factual del programa

### Experiencia del piloto (DDP 4.0)
- Dashboard como centro de mando
- Coach como briefing profesional continuo
- Explainability («¿por qué?»)
- Trayectoria / memoria deportiva
- Narrative Intelligence (voz de ingeniero de pista)
- Release Candidate auditada (RCA-1)
- **Project Memory System (PMS)** — memoria oficial del proyecto
- **Project Memory Infrastructure (PMI v1)** — `tools/pms/` · `/close-phase`
- **AI Context System (AICS v2)** — protocolo + ChatGPT Startup Prompt + `PROJECT_CONFIGURATION.md`

---

## Lo actual

```text
╔════════════════════════════════════════╗
║  DDP 4.0 · RELEASE CANDIDATE           ║
║  Producto offline usable               ║
║  Una historia por piloto activo        ║
║  PMS + PMI oficiales                   ║
╚════════════════════════════════════════╝
```

El piloto puede:

1. Importar telemetría  
2. Analizar e interpretar  
3. Incorporar al perfil  
4. Ver Dashboard / Coach / explicación  
5. Revisar su Trayectoria  

Sin consola. Sin depender de la memoria de un chat.

---

## Lo siguiente

### Coach en tiempo real
Acompañar al piloto **durante** la sesión, no solo al revisar.

### Ingeniero de pista
Leer la situación completa (entrenamiento, clasificación, carrera)
con la misma disciplina de evidencia que el producto offline.

Criterio de entrada: base RC estable (cumplido tras RCA-1).
Ejercitar E2E manual antes de ampliar alcance live.

---

## Lo futuro

### IA conversacional
Capa de comunicación sobre motores deterministas.
El piloto pregunta; DDP responde con la voz del sistema —
sin que la IA decida el programa.

### Mentor deportivo
Compañero de desarrollo a largo plazo: conoce la carrera del piloto,
explica el arco, acompaña decisiones de entrenamiento y competición.

---

## Visión (destino)

```text
Hoy
  Analiza · atribuye · programa · explica · recuerda

Después
  Coach en tiempo real
        ↓
  Ingeniero de pista
        ↓
  IA conversacional
        ↓
  Mentor deportivo
```

El destino no es «más análisis».
El destino es un **compañero de desarrollo** que conoce al piloto
y le ayuda a llegar más lejos de lo que llegaría solo.

Alineado con [`DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md) · Visión.

---

## Fuera de este roadmap (ingeniería aplazada)

Catálogo versionado, prioridad multi-fuente, `algorithm_version`,
más fuentes de import (carpeta iRacing, vigilancia, `.rpy`),
DriverIndex / informes career — ver `docs/ROADMAP.md` técnico.

---

*Actualizar al cerrar hitos de producto. No usar como backlog de tickets.*
