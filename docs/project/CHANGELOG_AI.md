# DDP — Changelog (visión de producto)

> Registro **funcional**. No técnico.
> Explica los cambios desde el punto de vista del piloto / producto.
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08

---

## 2026-08-08 — Prompt oficial de ChatGPT generado por el proyecto

DDP genera automáticamente el **ChatGPT Startup Prompt** a partir de
`PROJECT_CONFIGURATION.md`. Hay un **AI INITIALIZATION PROTOCOL** obligatorio.
Una conversación nueva puede empezar pegando ese prompt; la URL del repo de
documentación vive en un único sitio.

---

## 2026-08-08 — Una IA puede incorporarse en menos de un minuto

Existe el **AI Context System**: las inteligencias artificiales empiezan por
`AI_BOOTSTRAP.md` y solo consultan el resto del PMS cuando necesitan detalle.
Los humanos empiezan por `BOOTSTRAP.md`. El conocimiento vive en el proyecto,
no en los chats.

---

## 2026-08-08 — El PMS se mantiene como infraestructura

El Project Memory System ya no depende solo de la disciplina humana.
Existe una infraestructura oficial (`tools/pms/`) y el procedimiento
`/close-phase`: validar, actualizar memoria y solo entonces cerrar una fase —
igual de obligatorio que pasar los tests.

---

## 2026-08-08 — El proyecto recuerda su propia historia (PMS)

DDP incorpora el **Project Memory System**.
Cualquier desarrollador o conversación nueva puede recuperar el contexto
leyendo `docs/project/`, sin depender de chats anteriores.

---

## 2026-08-08 — Release Candidate estabilizado

Abrir el Dashboard ya no “deshace” quitar una sesión del perfil.
Borrar una telemetría recalcula la historia del piloto.
Settings permite elegir el piloto activo: Dashboard y Trayectoria cuentan
**una sola biografía**.
Si falta Attribution, la UI lo dice en lugar de inventar causas.

---

## 2026-08-08 — El Coach habla como ingeniero de pista

El briefing deja de sonar a estados de sistema.
Usa fechas, circuitos, curvas y contrastes Antes → Ahora.
Cierra siempre con un próximo paso accionable.
Orden: hecho → consecuencia → interpretación → próximo paso.

---

## 2026-08-08 — Trayectoria: cómo he llegado hasta aquí

La pantalla Piloto pasa a ser memoria deportiva:
capítulos (Inicio, Aprendizaje, Consolidación…) e hitos con evidencia.
Sin inventar podios ni skills validadas si no hay datos.

---

## 2026-08-08 — «¿Por qué?» con evidencia

El piloto puede abrir la explicación del foco:
por qué entrena eso, por qué no otra habilidad, con qué confianza
y cuándo cambiará el objetivo — sin hashes ni jerga interna.

---

## 2026-08-08 — Dashboard como centro de mando

En ~30 segundos responde: dónde estoy, si mejoro, qué entreno y qué hacer ahora.
Menos tarjetas KPI; un objetivo claro y un «qué debo hacer ahora» protagonista.

---

## 2026-08-08 — Strategy sustituye weaknesses

El programa del piloto ya no se elige mirando la primera debilidad de una lista.
**Development Strategy** decide el foco y por qué no otro entrenamiento.

---

## 2026-08-08 — Attribution explica el rendimiento

DDP separa dimensiones: deportiva, técnica, transferencia y lectura global.
Puede decir «evidencia insuficiente» en lugar de forzar una historia.

---

## 2026-08-08 — Entrenamiento anclado a curvas del circuito

El mismo skill (p. ej. control del acelerador) se entrena en las curvas
adecuadas de Bathurst, Spa, etc. — no un plan genérico vacío.

---

## 2026-08-08 — El Coach abre con el resultado deportivo

En clasificación y carrera, primero se reconoce el resultado
(victoria, podio, posición); después se explica el siguiente paso.
Una victoria no declara todas las skills consolidadas.

---

## 2026-08-08 — De hechos a consejo de sesión

Tras medir una sesión, DDP puede interpretar el problema prioritario,
proponer recomendaciones con criterio de éxito y mostrar un informe
regenerable — sin volver a medir la telemetría.

---

## 2026-08-08 — Medición objetiva de sesión

DDP mide ritmo, consistencia, errores y hallazgos por curva
de forma regenerable y determinista.

---

## 2026-08-08 — Importación de telemetrías iRacing

El piloto selecciona un `.ibt` y DDP lo convierte en sesión, vueltas
y telemetría persistidas, sin duplicar el mismo archivo.

---

## 2026-08-08 — Infraestructura local completa

DDP existe como aplicación de escritorio con base SQLite, migraciones
y capa de persistencia verificable.

---

*Añadir entradas nuevas arriba (más reciente primero) o al final por convención del equipo; mantener tono de producto.*
