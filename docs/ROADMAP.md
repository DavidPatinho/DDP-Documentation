# DDP — Roadmap

Punto de acumulación de lo que se ha decidido abordar más adelante.

Este documento **no modifica** `DATA_MODEL.md` v1.0 ni `DATABASE_DESIGN.md` v1.0,
ambos congelados. Recoge conceptos aceptados cuya implementación se aplaza, y sirve
para que una idea aprobada no se pierda entre fases. Nada de lo que hay aquí está
implementado.

Cuando uno de estos puntos exija cambiar el esquema, abre una versión 1.1 del
documento correspondiente con su propia auditoría, tal como fijan las reglas de uso
de ambos. Un apunte en este roadmap no autoriza por sí solo un cambio estructural.

Las cuatro decisiones pendientes del apartado 11 de `DATABASE_DESIGN.md` siguen
vivas y no se repiten aquí.

---

## Etapa cerrada: v0.4

**Estado:** motor de interpretación cerrado. Ver `CHANGELOG.md` y
`docs/INTERPRETATION_ENGINE.md` (v1.0 congelada).

Candidatos naturales posteriores: scoring / `DriverIndex`, informes semanal /
career, actualización de `TrackProfile`, o endurecimiento hacia v1.0.

Las etapas v0.1–v0.3 permanecen cerradas.

### Mejoras aceptadas (no bloqueantes) — v0.4

| Mejora | Notas | Cuándo |
|--------|-------|--------|
| Regresión: misma evidencia ⇒ mismas recomendaciones y mismo orden | Dos `SessionInterpretation` construidos a partir de la misma evidencia deben emitir recomendaciones idénticas y en el mismo orden (refuerzo de D14/D17) | Endurecimiento post-v0.4 |
| Regresión: unicidad de `primary_problem` | La misma evidencia no puede clasificarse simultáneamente bajo dos valores distintos de `primary_problem` (el coach ya elige uno; falta regresión explícita durable) | Endurecimiento post-v0.4 |
| Regresión: evidencia secundaria no altera el foco | Eliminar una evidencia secundaria no debe modificar la prioridad del diagnóstico salvo que esa evidencia fuese la responsable del `primary_problem` | Endurecimiento post-v0.4 |
| Regresión: presentación textual vs trazabilidad | Modificar únicamente el texto de presentación de una recomendación no debe alterar su trazabilidad (`evidence_refs`) ni su `success_criterion` | Endurecimiento post-v0.4 |
| Regresión: orden de construcción del request | Dos `InterpretationRequest` con la misma evidencia, construidos en distinto orden, deben producir el mismo `inputs_hash` y el mismo `SessionInterpretation` | Endurecimiento post-v0.4 |
| Regresión: reinterpretaciones concurrentes | Dos reinterpretaciones concurrentes sobre el mismo `Analysis` y la misma `engine_version` deben dejar un único `analysis_report` de sesión coherente (mismo `id` / `report_number`, hijas consistentes) | Endurecimiento operativo |
| Regresión: GET de interpretación sin efectos secundarios | Todas las rutas `GET` de interpretación deben permanecer libres de efectos secundarios y no modificar información persistida | Endurecimiento post-v0.4 |

---

## Etapa cerrada: v0.3

**Estado:** Core de análisis cerrado. Ver `CHANGELOG.md` y
`docs/ANALYSIS_ENGINE.md` (v1.0 congelada).

Etapa siguiente materializada: **v0.4 — Ingeniero IA** (también cerrada).

Las etapas v0.1 (infraestructura) y v0.2 (importación) permanecen cerradas.

### Mejoras aceptadas (no bloqueantes) — v0.3

| Mejora | Notas | Cuándo |
|--------|-------|--------|
| Regresión de análisis concurrentes misma sesión / `engine_version` | Garantizar resultado único y coherente bajo carrera en `AnalysisRepository.store` | Endurecimiento post-v0.3 |
| Regresión HTTP concurrente sobre la misma sesión | Determinismo + evitar recálculos redundantes cuando `inputs_hash` coincide | Tras Fase 6 / operación |
| Actualización automática de la UI al terminar un análisis con la vista abierta | Refresco reactivo sin recargar la página | Mejora de UX post-v0.3 |
| Comparación `external` cableada por series alineables en el Core | La API ya proyecta `external_lap`; el motor aún no mide por curva en ese modo | Cuando haya UX de selección (A5) |

---

## Etapa cerrada: v0.2

**Estado:** pipeline de importación iRacing cerrado. Ver `CHANGELOG.md` y
`docs/IMPORT_PIPELINE.md` (v1.0 congelada).

Siguiente candidato natural: **v0.3 — Core de análisis**, a partir de
`Session` + `TelemetryFile` ya persistidos.

La infraestructura de v0.1 permanece cerrada.

### Mejoras aceptadas (no bloqueantes)

| Mejora | Notas | Cuándo |
|--------|-------|--------|
| Ignorar canales IRSDK desconocidos / tipos nuevos sin tumbar la lectura | Refuerza compatibilidad con versiones futuras del simulador. El lector ya omite arrays y tipos no reconocidos; falta una comprobación de regresión explícita | Tras Fase 2, o en verificación E2E |
| Comprobar SHA-256 al recuperar un fichero del almacén | Verificación opcional de que el contenido sigue coincidiendo con el hash registrado | Al endurecer storage / verificación E2E |
| Regresión de atomicidad del pipeline ante fallo en persistencia | Forzar un error a mitad de la escritura y comprobar que la BD queda idéntica a antes del lote | Tras Fase 4 / verificación E2E |
| Métricas de ejecución por importación | Registrar duración, tamaño procesado y versiones del parser/Core en el resultado del lote para diagnóstico y rendimiento | Posterior a Fase 5 / endurecimiento operativo |
| Arrastrar y soltar `.ibt` en la UI | Misma cola y exactamente el mismo `POST /imports/ibt`; no nuevo pipeline | Mejora de UX post-v0.2 |

---

## 1. Versionado del catálogo

**Estado:** concepto aceptado. Sin implementar. Se retomará después de la fase de
importación.

### El problema

Los catálogos son la única categoría del esquema cuyo contenido viene de fuera y
cambia por su cuenta. Un circuito se renombra, una curva pasa a conocerse por otro
nombre, una serie cambia de denominación entre temporadas, aparece un coche nuevo.

Una instalación creada en 2026 tiene que poder recibir el catálogo de 2028 sin
duplicar lo que ya tiene, sin perder lo que el piloto haya creado o corregido a
mano, y sin romper las referencias que sus sesiones, vueltas, objetivos y perfiles
de circuito hayan establecido mientras tanto.

Es un problema distinto del versionado del esquema, y conviene no confundirlos: dos
instalaciones pueden compartir la misma versión de esquema y tener contenidos de
catálogo distintos.

### Preguntas que tendrá que responder

**Procedencia de cada fila.** Hoy `external_ids` dice qué plataforma conocía una
fila, pero no quién la creó. Distinguir entre lo que trae DDP, lo que llegó de una
importación y lo que escribió el piloto es la base de todo lo demás: sin esa
distinción, cualquier actualización corre el riesgo de pisar trabajo manual.

**Qué revisión de catálogo tiene una instalación.** Hace falta un marcador propio,
al lado de la versión de esquema y distinto de ella.

**Quién gana cuando discrepan.** Si el piloto renombró una curva y la actualización
la renombra de otra manera, hay que decidir. La política actual de `catalog.py`, que
rellena lo vacío y nunca sobrescribe lo que ya hay, es un punto de partida
razonable pero no una respuesta completa.

**Fusiones.** El caso caro: una actualización descubre que dos circuitos eran el
mismo. Fusionarlos exige reapuntar todas las referencias y decidir qué identidad
sobrevive. Es exactamente el escenario para el que existe el UUID.

**Bajas.** Un coche retirado de la plataforma no puede desaparecer de una
instalación que tiene sesiones corridas con él. El histórico manda sobre el
catálogo vigente.

### Lo que el esquema actual ya resuelve

Conviene reconocerlo para no rediseñar de más. Tres de los cinco frentes están
cubiertos sin tocar nada:

- Los **alias** absorben los renombrados sin mover ni una referencia. Es lo que
  hace que "Bathurst" y "Mount Panorama" convivan.
- Los **identificadores externos** enlazan una fila canónica con cualquier número
  de plataformas, así que un circuito compartido no se fragmenta.
- Los **atributos en JSON** absorben campos nuevos de una fuente sin migrar.
- El **UUID** da a cada fila fundamental una identidad estable e independiente del
  entero local, que es precisamente lo que necesita una fusión o una exportación.

### Lo que probablemente faltará

Dos añadidos, ambos aditivos: un marcador de procedencia por fila de catálogo y una
revisión de catálogo registrada junto a la versión de esquema. Ninguno obliga a
reconstruir tablas.

### Por qué se aplaza

Hasta que entren datos reales no hay nada que preservar, y las decisiones sobre
precedencia y fusión solo se pueden tomar bien viendo cómo llegan los datos de
verdad. Diseñarlo antes sería adivinar.

---

## 2. Prioridad por origen de datos

**Estado:** concepto aceptado. Sin implementar. Se retomará cuando exista más de una
fuente oficial.

### El problema

Las fuentes se contradicen, y no es un caso hipotético: es lo que ya ocurre. [I] y [W]
publican iRating distinto para la misma sesión, y las unicidades del esquema obligan a
que solo uno sobreviva. La fase 3 lo resuelve por orden de importación: gana el
primero que entró, y para cambiarlo hay que pedirlo de forma explícita.

Es una regla válida y tiene una virtud que conviene no perder: **ninguna importación
puede pisar en silencio un dato existente**. Pero decide por accidente. Con dos
fuentes oficiales, el ganador depende de en qué orden se ejecutaron las
importaciones, que es justo lo que no debería determinar la verdad.

### Preguntas que tendrá que responder

**Jerarquía entre fuentes.** El resultado oficial del simulador debería pesar más que
un informe redactado a partir de él. Hoy no hay forma de expresarlo.

**Granularidad.** Si la prioridad es por fuente entera o por campo. Una fuente puede
ser la mejor para el rating y peor para el detalle de vueltas, y eso solo se puede
aprovechar con prioridad por campo.

**El lugar de la corrección manual.** Lo que escribe el piloto tiene que estar por
encima de cualquier fuente automática, y no puede perderse en la siguiente
importación. Es el mismo problema de procedencia que plantea el versionado del
catálogo, y conviene resolver los dos con el mismo mecanismo.

**Auditoría de quién ganó.** Si un valor se sustituye por venir de una fuente de más
rango, hay que poder explicar por qué. Sin ese rastro, un dato que cambia sin
intervención humana es indistinguible de un error.

### Lo que el esquema actual ya resuelve

`import_batch` guarda la fuente y el registro original de cada importación, y cada
sesión referencia el lote que la trajo, así que la procedencia a nivel de fila ya
existe. Lo que falta es procedencia **por valor** y un orden entre fuentes.

### Por qué se aplaza

Con una sola fuente oficial la regla actual no se equivoca nunca. El diseño de la
jerarquía necesita conocer las fuentes reales y en qué se contradicen de verdad, y
eso solo se ve importando.

---

## 3. Trazabilidad del motor de análisis

**Estado:** concepto aceptado. Sin implementar. Se retomará cuando el Core tenga
más de un algoritmo de cálculo y parámetros configurables.

### El problema

Hoy cada fila derivada lleva `engine_version`, `computed_at` e `inputs_hash`. Eso
responde a *qué motor*, *cuándo* y *sobre qué entradas*, y es suficiente mientras
exista una sola fórmula por versión.

Cuando el motor crezca, esa tríada se queda corta. La misma versión del motor
puede aplicar algoritmos distintos (por ejemplo, dos formas de repartir la pérdida
por curva) o los mismos algoritmos con parámetros distintos (umbrales, pesos,
ventana de consistencia). Mezclarlos bajo un único `engine_version` haría
imposible saber por qué dos análisis de la misma versión no coinciden.

### Lo que se propone

Separar la trazabilidad en tres capas, sin tocar todavía el esquema:

| Campo | Responde a |
|-------|------------|
| `engine_version` | Qué entrega del Core produjo el resultado |
| `algorithm_version` | Qué algoritmo concreto dentro de esa entrega |
| `parameters_hash` | Con qué parámetros se ejecutó |

`inputs_hash` se mantiene: es la huella de las **entradas** (vueltas, telemetría),
no de la configuración del cálculo. Las tres nuevas cubren la configuración; la
cuarta, los datos.

### Relación con la frontera medición / interpretación

La fase 4 fijó que `analysis` mide y no opina. Esta trazabilidad refuerza esa
frontera: si un número cambia, tiene que poder decirse si cambió porque cambiaron
las entradas, el algoritmo o los parámetros. Sin eso, una diferencia entre dos
análisis es indistinguible de un bug.

### Por qué se aplaza

Con un solo algoritmo y sin parámetros configurables, `engine_version` +
`inputs_hash` ya hacen verificable la reconstrucción. Añadir columnas ahora sería
inventar una forma antes de conocer los algoritmos reales. Cuando llegue, será un
cambio aditivo (v1.1 del diseño) y no obliga a reconstruir filas existentes.

---

## 4. Otros puntos abiertos

### Presupuesto de índices

El apartado 6.4 de `DATABASE_DESIGN.md` presupuesta "alrededor de 30 índices en
total" para las 41 tablas. Al cerrar la fase 3 hay **50 sobre 25 tablas**: 30 de los
catálogos, 3 del piloto y 17 de importación y sesión.

No hay índices de más. Todos salen de reglas del propio diseño: índice único por
UUID (D1), por clave natural (5.1), por `(source, external_id)` (D6), índice en cada
clave ajena hija (10.1) y los que el apartado 6 nombra uno por uno. Lo que se queda
corto es la cifra, no la disciplina.

Parte de la diferencia es contable, porque se han usado índices con nombre en lugar
de restricciones `UNIQUE` en línea, y así son visibles al inventariar. La parte que
no es contable viene de que el apartado 6 razona sobre las consultas de la interfaz
y el 10.1 exige además cubrir cada clave ajena hija, que son dos criterios
distintos aplicados a la misma tabla.

La fase 9 es donde se miden y se podan, con volumen real. Quitar un índice es una
migración aditiva y reversible, así que no urge, pero la cifra del documento habrá
que corregirla en una futura 1.1.

---

## 5. Fuentes de importación

**Estado:** concepto aceptado. Sin implementar. Explicitamente fuera del alcance
de v0.2. Se retomará en una versión posterior.

### El problema

Hoy el pipeline de v0.2 importa telemetrías a partir de un archivo `.ibt`
seleccionado por el usuario. Eso basta para cerrar el contrato
`bytes → TelemetryImport → persistencia`, pero no cubre cómo el piloto llega a
esos bytes en el día a día: carpeta de iRacing, arrastrar y soltar, vigilancia
automática al terminar una carrera, o el replay asociado.

### Evolución prevista

DDP deberá soportar **múltiples fuentes de importación** utilizando exactamente
el mismo pipeline ya diseñado en `IMPORT_PIPELINE.md`.

Posibles fuentes:

- Selección manual de un archivo `.ibt`
- Importación directa desde la carpeta de documentos de iRacing
- Arrastrar y soltar archivos sobre DDP
- Vigilancia automática de la carpeta de telemetrías de iRacing para importar
  nuevas sesiones al finalizar una carrera
- Asociación automática del `.rpy` (replay) correspondiente cuando exista

### Requisitos que fijan el diseño futuro

- El **Core no cambia**
- El **orquestador no cambia**
- Todas las fuentes terminan entregando el **mismo flujo de importación**
- La detección automática es únicamente una **nueva fuente de entrada** al
  pipeline existente (obtiene bytes / rutas y los entrega al Backend)

No se reabre el contrato `TelemetryImport` ni la frontera Core / Backend para
acomodar estas entradas. Solo se añaden adaptadores de origen delante del
pipeline.

### Por qué se aplaza

v0.2 debe demostrar primero que un `.ibt` entra, se normaliza y se persiste sin
duplicar hechos. Las fuentes adicionales son producto y UX sobre ese cimiento;
diseñarlas antes mezclaría el cierre del pipeline con decisiones de shell,
permisos de carpeta y ciclo de vida de la aplicación.

---

## Estado

| # | Punto | Fase prevista |
|:-:|-------|---------------|
| 1 | Versionado del catálogo | Después de la importación |
| 2 | Prioridad por origen de datos | Cuando haya varias fuentes oficiales |
| 3 | Trazabilidad del motor (`engine_version` · `algorithm_version` · `parameters_hash`) | Cuando el Core tenga varios algoritmos |
| 4 | Revisión del presupuesto de índices | Fase 9 |
| 5 | Fuentes de importación (carpeta iRacing, DnD, vigilancia, `.rpy`) | Posterior a v0.2 |
