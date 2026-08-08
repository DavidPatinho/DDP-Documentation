# DDP — Diseño de la Base de Datos

**Versión 1.0 · Congelado · Referencia oficial para implementar SQLite**

> Documento derivado **exclusivamente** de `docs/DATA_MODEL.md` v1.0.
>
> No contiene SQL, esquema ejecutable ni código. Diseña la persistencia antes de
> implementarla.
>
> El modelo de dominio no se modifica. Ninguna entidad nueva, ningún atributo de
> dominio nuevo, ninguna relación nueva.
>
> **Congelado durante la implementación.** Ninguna modificación estructural se hace
> sobre la marcha. Si al implementar SQLite aparece una necesidad estructural, el
> desarrollo se detiene y se abre una versión 1.1 de este documento con su
> justificación. Cambiar el esquema mientras se programa es lo que convierte un
> diseño en una acumulación de parches.

---

# 1. Alcance y frontera con el modelo

Traducir un modelo de dominio a una base de datos exige añadir cosas que el
dominio no conoce: claves internas, marcas de tiempo técnicas, huellas de
integridad, versiones de esquema. Para que quede claro que el modelo sigue
congelado, esta es la frontera exacta.

## Lo que este diseño **sí** puede añadir

| Añadido | Motivo |
|---------|--------|
| Claves primarias internas | El modelo pide un identificador técnico opaco; aquí se elige su forma física |
| `created_at` · `updated_at` | Auditoría técnica de filas, no información de dominio |
| Huellas de contenido e integridad | Evitar duplicados en la importación |
| Estado de disponibilidad de un fichero en disco | El fichero vive fuera de la base de datos; su presencia es un hecho de infraestructura |
| Procedencia de importación | Poder auditar, repetir o revertir una importación |
| Tablas de esquema, ajustes y migraciones | Infraestructura de la aplicación |
| Tablas hijas y tablas puente | Forma física de las listas y las relaciones N↔M del modelo |

## Lo que este diseño **no** puede añadir

Atributos de dominio, entidades, relaciones, ni cambios de clasificación. Si al
implementar aparece la necesidad de alguno, **no se resuelve aquí**: se abre una
versión 1.1 de `DATA_MODEL.md` con su auditoría, y este documento se actualiza
después.

## Regla de derivación

```
DATA_MODEL.md v1.0  ──>  DATABASE_DESIGN.md  ──>  SQLite
   (qué existe)          (cómo se guarda)        (implementación)
```

La flecha nunca va hacia la izquierda. Una dificultad de implementación no
justifica cambiar el diseño, y una dificultad de diseño no justifica cambiar el
modelo.

---

# 2. Filosofía de la persistencia

## 2.1 La distinción que gobierna todo

El modelo separa hechos de derivados. En la base de datos esa separación deja de
ser conceptual y se vuelve operativa: determina qué se puede borrar sin pensar y
qué no se puede perder nunca.

```
PERMANENTE          Ocurrió. No se corrige, no se recalcula, no se borra.
                    Si se pierde, no hay forma de recuperarlo.
                          │
                          ▼
REGENERABLE         Se calculó a partir de lo permanente.
                    Se puede borrar entero y reconstruir.
```

## 2.2 Datos permanentes

Son la única cosa que una copia de seguridad tiene que proteger.

| Qué | Por qué es permanente |
|-----|----------------------|
| `session` y sus métricas | Es lo que pasó en pista. Una carrera no se recalcula |
| `lap` | Los tiempos son el registro histórico del progreso |
| `telemetry_file` (metadatos) | Es el puntero al registro crudo del que todo se deriva |
| `milestone` | Logro certificado por un tercero. Irreproducible |
| `objective` (definición) | La escribió una persona o un informe. No hay fórmula que la reconstruya |
| `driver` | Identidad y hardware |
| Catálogos referenciados por hechos | Sin ellos, los hechos pierden significado |

## 2.3 Datos derivados

Producidos por el Core a partir de los permanentes.

| Qué | Se reconstruye desde |
|-----|----------------------|
| `analysis` y sus hijas | `session` + `telemetry_file` |
| `driver_index` y sus dimensiones | `analysis` + histórico |
| `analysis_report` y sus hijas | `analysis` + `objective` |
| `track_profile` y sus hijas | Todas las sesiones del combo |
| `career` y sus hijas | Todo el histórico del piloto |
| `objective_progress` | `objective` + sesiones posteriores a su alta |

## 2.4 Datos regenerables: dos grados, no uno

Aquí el diseño tiene que ser más fino que el modelo, porque no todo lo derivado
se puede tratar igual.

**Desechable.** Se puede vaciar la tabla entera y reconstruirla con
identificadores nuevos, porque nada fuera de la capa derivada la referencia:
`analysis`, `driver_index`, `track_profile`, `career`, `objective_progress`.

**Regenerable en el sitio.** Su contenido se recalcula pero **su identidad se
conserva**: `analysis_report`.

Hay dos razones para tratar `analysis_report` de forma distinta. La primera es
estructural: `objective.source_analysis_report_id` es la **única** clave ajena de
todo el esquema que va de una tabla permanente a una derivada, así que vaciar los
informes rompería la procedencia de los objetivos. La segunda es humana: un
informe es un documento que el piloto leyó. Sustituirlo por otro con otro
identificador borra la historia de lo que se le dijo y cuándo.

Consecuencia de diseño: recalcular un informe **actualiza la fila existente** y
conserva su `report_number` y su identificador. Nunca se borra para volver a
crearlo.

## 2.5 Datos que nunca deben eliminarse

| Nunca | Ni siquiera cuando |
|-------|--------------------|
| `session`, `lap` y sus métricas | El usuario quiera "limpiar" el historial. Se archiva, no se borra |
| `milestone` y su certificado | Se borre la sesión que lo originó. La sesión se protege por eso |
| `telemetry_file` (la fila) | El fichero de disco se archive o desaparezca. La fila registra que existió |
| `objective` (la definición) | Esté cumplido o abandonado. Es la historia de las intenciones del piloto |
| El fichero de certificado en disco | Nunca. No es regenerable |

## 2.6 Cuatro principios operativos

**El fichero de telemetría nunca entra en la base de datos.** Decenas de
gigabytes viven en disco; la base de datos guarda metadatos y una ruta. Es la
decisión que mantiene la base de datos en el orden de los cientos de megabytes
después de diez años.

**Un dato que puede calcularse no se almacena como hecho.** Regla heredada del
modelo. En persistencia significa: si un valor está en una tabla derivada y
también en una permanente, hay un error de diseño.

**Lo derivado se puede tirar sin miedo.** Es lo que permite cambiar el motor de
análisis sin migrar datos.

**Toda importación es atómica.** Una importación a medias es peor que ninguna: un
fallo debe dejar la base de datos exactamente como estaba.

---

# 3. Estrategia general: clasificación de entidades

El modelo clasifica en tres capas por su naturaleza. La persistencia necesita seis
categorías, porque lo que determina cómo se guarda un dato no es solo si es un
hecho, sino también su volumen, quién lo escribe y cuánto cambia.

## 3.1 Catálogos

**Simulator · Car · Track · Corner · Series · Season**

| Rasgo | Valor |
|-------|-------|
| Volumen | Bajo: decenas a pocos miles de filas |
| Escritura | Rara, casi siempre por importación |
| Ciclo de vida | Se crean una vez y sobreviven a todo |
| Compartidos | Sí: los mismos circuitos y coches sirven para cualquier piloto |

**Motivo de la clasificación.** No describen lo que hizo el piloto sino el mundo
en el que lo hizo. Son referencia estable, se pueden sembrar de antemano, y su
integridad es crítica porque todos los hechos apuntan a ellos. Por eso ninguna
operación de usuario puede borrar un catálogo que esté en uso.

`Corner` está aquí y no en datos de sesión, aunque parezca análisis, porque la
curva es una propiedad del circuito. Lo que depende del piloto es su
clasificación como fuerte o débil, y eso vive en `track_profile`.

## 3.2 Datos de usuario

**Driver · Objective · Milestone**

| Rasgo | Valor |
|-------|-------|
| Volumen | Muy bajo: decenas a cientos de filas |
| Escritura | Manual o semiautomática |
| Ciclo de vida | Vive tanto como el piloto use la aplicación |
| Regenerable | **No** |

**Motivo de la clasificación.** Es lo único que no se puede reconstruir desde
ninguna otra fuente. Un objetivo lo escribió alguien; un hito lo certificó
iRacing; el hardware lo declaró el piloto. Si se pierde esta categoría, la
aplicación sigue teniendo datos pero pierde la intención y el reconocimiento, que
es justamente lo que la hace personal.

`Objective` es de usuario aunque a veces lo proponga un informe: la propuesta se
convierte en un registro autorizado en el momento en que se acepta.

## 3.3 Datos de sesión

**Session · Lap · TelemetryFile**

| Rasgo | Valor |
|-------|-------|
| Volumen | **Alto**: miles de sesiones, cientos de miles de vueltas |
| Escritura | Por lotes, en importación |
| Ciclo de vida | Inmutable tras la importación |
| Regenerable | Solo reimportando desde la fuente original |

**Motivo de la clasificación.** Es la categoría que dicta el rendimiento de todo
el sistema, porque es la única que crece sin límite. Se separa de los datos de
usuario porque necesita un tratamiento distinto en tres frentes: se escribe por
lotes y no fila a fila, se consulta siempre paginada y ordenada, y es la única
donde el ancho de la fila importa de verdad.

Se separa de los catálogos porque es inmutable y de altísima cardinalidad: un
catálogo se actualiza, una sesión jamás.

## 3.4 Datos derivados

**Analysis · DriverIndex · AnalysisReport · TrackProfile · Career**
(más `objective_progress`)

| Rasgo | Valor |
|-------|-------|
| Volumen | Medio: proporcional a las sesiones, no a las vueltas |
| Escritura | Automática, por el Core |
| Ciclo de vida | Vinculado a la versión del motor que lo produjo |
| Regenerable | **Sí**, entero |

**Motivo de la clasificación.** Son cachés con nombre de dominio. Se agrupan
porque comparten tres necesidades: llevan trazabilidad de qué versión del motor
los generó, se pueden invalidar en bloque, y se pueden reconstruir sin consultar
ninguna fuente externa.

Esta categoría es también **la estrategia de rendimiento del sistema**, no un
subproducto. `track_profile` y `career` existen precisamente para que el
dashboard no tenga que recorrer 500.000 vueltas.

## 3.5 Configuración

**Ninguna entidad del modelo.**

Esto es intencionado y conviene explicarlo, porque la ausencia parece un olvido.
La auditoría del modelo sacó de `Driver` los atributos `current_car_id` y
`current_season_label` con el argumento de que eran estado de interfaz y no
dominio. Fueron a ninguna parte: siguen siendo necesarios, pero no son dominio.

Su sitio es esta categoría, formada por tablas técnicas que no representan
entidades:

| Tabla | Contenido |
|-------|-----------|
| `app_setting` | Piloto activo, tema, rutas de importación, preferencias de interfaz |
| `schema_migration` | Historial de migraciones aplicadas |
| `import_batch` | Procedencia de cada importación y su registro original |

| Rasgo | Valor |
|-------|-------|
| Volumen | Trivial |
| Regenerable | Parcialmente: las preferencias se pueden perder sin drama |
| Copia de seguridad | Deseable, no crítico |

## 3.6 Futuras

**Setup · Sector · Team**

No se crea ninguna tabla. El modelo las reserva sin atributos, y crear tablas
vacías solo genera confusión sobre si están en uso.

**Motivo.** Las tres llegarán por integración con Garage61, y su forma dependerá
de lo que esa integración entregue realmente. Diseñarlas ahora sería adivinar. La
sección 8 explica por qué añadirlas más adelante no rompe nada.

## 3.7 Resumen

| Categoría | Entidades | Volumen | Regenerable | Prioridad de copia |
|-----------|:---------:|:-------:|:-----------:|:------------------:|
| Catálogos | 6 | Bajo | Por importación | Alta |
| Datos de usuario | 3 | Muy bajo | **No** | **Máxima** |
| Datos de sesión | 3 | **Alto** | Por reimportación | **Máxima** |
| Datos derivados | 5 | Medio | **Sí** | Ninguna |
| Configuración | — | Trivial | Parcial | Baja |
| Futuras | 3 | — | — | — |

La columna de la derecha es la conclusión práctica de todo el apartado: una copia
de seguridad que incluya usuario, sesión y catálogos, y omita los derivados, es
completa y pesa una fracción.

---

# 4. Decisiones de traducción

El modelo contiene seis construcciones que no tienen una forma física evidente.
Resolverlas antes de diseñar tablas evita rehacer el esquema más tarde.

## 4.1 Criterio general: tabla hija o columna JSON

El modelo está lleno de listas (`aliases[]`, `metrics[]`, `sections[]`,
`channels[]`). No todas merecen una tabla.

| Se usa tabla hija cuando | Se usa columna JSON cuando |
|--------------------------|----------------------------|
| Los valores se filtran, ordenan o agregan | La lista se lee siempre completa con su fila padre |
| Los valores apuntan a otra entidad y necesitan clave ajena | No hay referencias a otras tablas |
| La lista puede crecer sin límite | La lista es corta y de tamaño acotado |
| Hay que indexar por sus valores | No se consulta nunca por su contenido |

Aplicación del criterio:

| Lista del modelo | Forma física | Motivo |
|------------------|--------------|--------|
| `aliases[]` de Car, Track, Corner | **Tabla hija** | Se buscan al importar, para reconocer "Bathurst" como Mount Panorama |
| `Session.metrics[]` | **Tabla hija** | Se agregan para el gráfico de evolución |
| `Session.flags[]` | **JSON** | Corta y de vocabulario abierto. Filtrar 5.000 sesiones sin índice es gratuito |
| `Season.schedule[]` | **Tabla hija** | Apunta a circuito y coche con clave ajena |
| `DriverIndex.dimensions[]` | **Tabla hija** | Se comparan entre fotografías y se grafican por dimensión |
| `Analysis.time_loss_by_corner[]` | **Tabla hija** | Es la consulta "análisis por curva" |
| `Analysis.estimated_gains[]` | **Tabla hija**, la misma que la pérdida por curva | Mismo grano: un hecho por análisis y curva |
| Hijas de AnalysisReport | **Tabla hija** | Ordenadas, con texto largo y referencias a curvas |
| `TrackProfile.corner_assessments[]` | **Tabla hija** | Apunta a curva |
| Fortalezas y debilidades de Career | **JSON** | Siete valores, sin claves ajenas, leídos con su fila padre |
| `Simulator.rating_metrics[]` · `telemetry_formats[]` | **JSON** | Metadatos declarativos, se leen enteros |
| `Track.zones[]` | **JSON** | Una sola zona observada, nunca se filtra |
| `TelemetryFile.channels[]` | **JSON** | Se lee con el fichero, no se consulta por canal |
| `Analysis.errors_detected[]` | **JSON** | Observaciones de texto sin referencias |

## 4.2 D1 · Forma física del identificador

El modelo exige "identificador técnico opaco (UUID)". En SQLite eso tiene un
coste medible, porque cada clave ajena y cada índice repite el identificador.

| Opción | Coste en `lap` (500.000 filas) | Portabilidad |
|--------|-------------------------------|--------------|
| UUID como texto de 36 caracteres | ~18 MB por columna indexada | Alta |
| UUID como BLOB de 16 bytes | ~8 MB por columna indexada | Alta |
| Entero interno | ~2,5 MB por columna indexada | Nula entre instalaciones |

**Recomendación.** Las dos cosas, con papeles separados:

- **Clave primaria `INTEGER`** en todas las tablas. En SQLite el `rowid` existe de
  todas formas, así que usarlo como clave primaria es gratis y hace que todos los
  cruces se hagan sobre enteros de 8 bytes.
- **UUID como BLOB de 16 bytes, con índice único**, solo en las **12 entidades
  fundamentales**. Es el identificador opaco que pide el modelo, sirve para
  exportar y para fusionar dos instalaciones en el futuro, y no se paga en las
  tablas hijas ni en las derivadas.

Las tablas derivadas y las hijas no llevan UUID: son regenerables y nadie las
referencia desde fuera de la base de datos.

Si se usa UUID versión 7, los identificadores quedan ordenados por tiempo de
creación, lo que mejora la localidad del índice al importar por lotes.

## 4.3 D2 · `attributes{}` como columna JSON

Los datos específicos de una plataforma que el modelo envía a `attributes{}` van a
una **columna JSON** en la propia tabla de la entidad.

**Motivo.** Su forma es desconocida por definición: es el mecanismo que existe
precisamente para no tener que migrar cuando llega una fuente nueva. Una tabla de
pares clave-valor daría lo mismo con más complejidad y más cruces.

**Camino de crecimiento**, y es la parte importante:

```
1. El dato llega y vive dentro de attributes           coste: cero
2. Se consulta a menudo  ->  índice sobre la expresión coste: un índice
3. Es central en el dominio  ->  version 1.1 del modelo y columna propia
```

Los tres pasos son aditivos. Ninguno obliga a reconstruir tablas ni rompe
instalaciones existentes.

## 4.4 D3 · `raw_payload` por lote de importación

El modelo pide conservar el registro original tal como llegó. Guardarlo en cada
fila no es viable: 500.000 vueltas con un registro de origen de medio kilobyte
son unos 250 MB de texto que casi nunca se lee, más que todo el resto de la base
de datos junta.

**Recomendación.** Una tabla técnica `import_batch` guarda **una** fila por
operación de importación, con la respuesta original completa. Cada fila importada
lleva su `import_batch_id`.

Esto cumple el propósito de M3, que es no perder información y poder reinterpretar
después lo que hoy no se sabe leer, y además regala tres cosas: auditoría de qué
entró y cuándo, posibilidad de repetir una importación, y posibilidad de revertirla
por completo. El registro original se puede archivar pasado un tiempo sin tocar
ningún hecho.

## 4.5 D4 · `TelemetryFile` sin polimorfismo

El modelo dice que un fichero de telemetría puede referirse a una sesión completa
o a una sola vuelta. La solución habitual sería una referencia polimórfica, que en
SQLite significa renunciar a la clave ajena y abrir la puerta a filas huérfanas.

No hace falta. **Toda vuelta pertenece a una sesión**, incluidas las vueltas
externas de Garage61, que el modelo almacena como vueltas de una sesión de un
piloto externo. Por tanto la sesión se conoce siempre.

| Columna | Nulo | Significado |
|---------|:----:|-------------|
| `session_id` | No | Siempre presente. Clave ajena real |
| `lap_id` | Sí | Presente solo si el fichero es de una vuelta concreta |

El `scope` del modelo **no se almacena**: se deduce de si `lap_id` está vacío. Es
la propia regla del modelo aplicada a sí misma, un dato que puede calcularse no se
guarda como hecho.

Resultado: dos claves ajenas verificables, cero polimorfismo, cero huérfanos
posibles.

## 4.6 D5 · `Objective` partido en dos tablas

El modelo marca `Objective` como fundamental en su definición y derivado en su
progreso. La persistencia respeta esa línea físicamente:

| Tabla | Naturaleza | Contenido |
|-------|-----------|-----------|
| `objective` | Permanente | Enunciado, naturaleza, valor objetivo, ámbito, prioridad, procedencia, refinamiento |
| `objective_progress` | Derivada, 1:1 | Estado, porcentaje, valor actual, hito de cierre, fecha de evaluación |

**Motivo.** Si el progreso viviera en la misma fila, "vaciar todo lo derivado"
implicaría modificar filas de una tabla permanente, y la operación dejaría de ser
segura. Partido en dos, se puede vaciar `objective_progress` entero y
reconstruirlo sin tocar una sola definición.

## 4.7 D6 · `external_ids[]` con una tabla por entidad

Es la decisión más importante del diseño en materia de integridad.

| Opción | Tablas | Clave ajena | Huérfanos |
|--------|:------:|:-----------:|:---------:|
| Una tabla compartida con discriminador de tipo | 1 | **Imposible** | Inevitables con el tiempo |
| Una tabla por entidad | 7 | Real y verificada | Imposibles |

**Recomendación: una tabla por entidad.** Siete tablas de forma idéntica
—`driver_external_id`, `car_external_id`, `track_external_id`,
`series_external_id`, `season_external_id`, `session_external_id`,
`lap_external_id`— cada una con su clave ajena verificada por el motor.

Una tabla compartida con una columna `entity_type` parece más limpia y es la
trampa clásica: el motor no puede validar nada, y basta un borrado mal hecho para
dejar identificadores externos apuntando a filas que ya no existen. El requisito de
integridad de este documento sería imposible de cumplir.

Solo se crean para las siete entidades que las fuentes identifican realmente.
`Simulator` es la fuente, y a `Corner` no le asigna identificador ni iRacing ni
Garage61. Añadir una tabla más adelante es una migración puramente aditiva.

**Índice único `(source, external_id)`** en cada una. Es lo que hace imposible
importar dos veces la misma subsesión.

---

# 5. Diseño lógico

**41 tablas**: 14 de catálogo, 5 de usuario, 6 de sesión, 13 de derivados y 3
técnicas.

Convenciones del inventario. *PK* es la clave primaria interna; todas son enteras
según D1. *Única* es la restricción que impide duplicados al importar.
*Depende de* enumera las claves ajenas salientes. Las tablas hijas se listan
debajo de su padre con sangría.

## 5.1 Catálogos

| Entidad | Tabla | PK | Única | Depende de | Cardinalidad |
|---------|-------|----|-------|------------|--------------|
| Simulator | `simulator` | `id` | `name` | — | 1 → N series, sesiones |
| Car | `car` | `id` | `name` | — | 1 → N sesiones, perfiles |
| Track | `track` | `id` | `name` | — | 1 → N curvas, sesiones, perfiles |
| Corner | `corner` | `id` | `(track_id, name)` | `track` | N → 1 circuito |
| Series | `series` | `id` | `(simulator_id, name)` | `simulator` | 1 → N temporadas |
| Season | `season` | `id` | `(series_id, year, season_number)` | `series` | 1 → N sesiones |

Tablas hijas:

| Tabla | Padre | Única | Notas |
|-------|-------|-------|-------|
| `car_alias` | `car` | `alias` | Único global: un alias no puede señalar a dos coches |
| `track_alias` | `track` | `alias` | Ídem. Resuelve "Bathurst" al importar |
| `corner_alias` | `corner` | `(track_id, alias)` | Resuelve "Forest Elbow" |
| `season_schedule_entry` | `season` | `(season_id, iso_week)` | Depende también de `track` y `car` |
| `car_external_id` | `car` | `(source, external_id)` | |
| `track_external_id` | `track` | `(source, external_id)` | |
| `series_external_id` | `series` | `(source, external_id)` | |
| `season_external_id` | `season` | `(source, external_id)` | |

`simulator` lleva en JSON sus métricas de progresión y sus formatos de telemetría.
`track` lleva en JSON sus zonas.

**Dependencias.** Los catálogos solo dependen entre sí, en una jerarquía sin
ciclos: `simulator → series → season` y `track → corner`. Ninguno depende de datos
de usuario, de sesión ni de derivados. Son la base del grafo.

## 5.2 Datos de usuario

| Entidad | Tabla | PK | Única | Depende de | Cardinalidad |
|---------|-------|----|-------|------------|--------------|
| Driver | `driver` | `id` | `uuid` | — | 1 → N sesiones, objetivos, hitos |
| Milestone | `milestone` | `id` | `(driver_id, title)` | `driver`, `session`, `series`, `car`, `track` | N → 1 piloto |
| Objective | `objective` | `id` | — | `driver`, `corner`, `analysis_report`, `milestone`, `objective` | N → 1 piloto |

Tablas hijas:

| Tabla | Padre | Única | Notas |
|-------|-------|-------|-------|
| `driver_external_id` | `driver` | `(source, external_id)` | Identidad por plataforma |
| `objective_progress` | `objective` | `objective_id` | Derivada, 1:1. Ver D5 |

El hardware del piloto va en columnas de `driver`: es una estructura 1:1 siempre
presente y de cuatro campos, y una tabla aparte solo añadiría un cruce. Si algún
día hace falta historial de hardware, extraerla es una migración aditiva.

`objective` **no tiene clave natural**, tal como advierte el modelo: los enunciados
se repiten refinados entre periodos, así que "Bajar de 2:30" puede existir dos
veces legítimamente. La cadena de refinamiento se distingue por `supersedes_id`.

**Dependencias y su rareza.** `objective.source_analysis_report_id` es la **única
clave ajena de todo el esquema que va de una tabla permanente a una derivada**. Su
existencia es la razón por la que `analysis_report` se regenera en el sitio y no se
vacía (apartado 2.4). Lleva además `ON DELETE SET NULL` como red de seguridad: si
alguna vez desaparece un informe, el objetivo sobrevive y solo pierde la
procedencia.

## 5.3 Datos de sesión

| Entidad | Tabla | PK | Única | Depende de | Cardinalidad |
|---------|-------|----|-------|------------|--------------|
| Session | `session` | `id` | `(driver_id, started_at)` | `simulator`, `driver`, `track`, `car`, `season`, `import_batch` | 1 → N vueltas, ficheros |
| Lap | `lap` | `id` | `(session_id, lap_number)` | `session` | N → 1 sesión |
| TelemetryFile | `telemetry_file` | `id` | `content_hash` | `session`, `lap` | N → 1 sesión, N → 0..1 vuelta |

Tablas hijas:

| Tabla | Padre | Única | Notas |
|-------|-------|-------|-------|
| `session_metric` | `session` | `(session_id, name)` | Mecanismo M4. Única fuente de verdad del rating |
| `session_external_id` | `session` | `(source, external_id)` | La subsesión del simulador |
| `lap_external_id` | `lap` | `(source, external_id)` | Solo para vueltas de origen externo. Tabla dispersa |

`session` y `telemetry_file` llevan columna JSON `attributes` para condiciones,
meteorología, nivel de parrilla y metadatos de fichero. `telemetry_file` lleva en
JSON sus canales, y `session` lleva en JSON sus banderas.

**Las banderas no incluyen "Pole".** Es exactamente `start_position = 1`, que ya
es una columna de `session`, y guardarlo dos veces incumpliría la regla de no
almacenar como hecho lo que puede calcularse. Solo se registran las banderas no
derivables, como "Fast Lap", que depende de las vueltas de los demás.

**`lap_external_id` es dispersa, no paralela a `lap`.** Las vueltas propias se
identifican por la subsesión de su sesión más su número; solo necesitan
identificador externo las que llegan de una fuente externa, es decir las vueltas
de referencia de Garage61. Son miles de filas, no cientos de miles.

**Cardinalidad real esperada** con diez años de uso: 5.000 sesiones, 500.000
vueltas, 10.000 métricas de sesión, 5.000 ficheros de telemetría de sesión más los
de vuelta que se importen de Garage61.

**Dependencias.** Toda la categoría cuelga de los catálogos y del piloto. Nada de
aquí depende de derivados, lo cual es la propiedad que permite vaciar la capa
derivada entera sin consecuencias.

## 5.4 Datos derivados

| Entidad | Tabla | PK | Única | Depende de | Cardinalidad |
|---------|-------|----|-------|------------|--------------|
| Analysis | `analysis` | `id` | `(session_id, engine_version)` | `session`, `lap` | 1 → 1 sesión por versión |
| DriverIndex | `driver_index` | `id` | `(driver_id, computed_at, kind)` | `driver`, `analysis` | N → 1 piloto |
| AnalysisReport | `analysis_report` | `id` | `(scope, report_number)` | `driver`, `season`, `driver_index` | N ↔ M sesiones |
| TrackProfile | `track_profile` | `id` | `(driver_id, track_id, car_id)` | `driver`, `track`, `car`, `lap` | 1 por combinación |
| Career | `career` | `id` | `driver_id` | `driver` | 1 → 1 piloto |

Tablas hijas:

| Tabla | Padre | Única | Notas |
|-------|-------|-------|-------|
| `analysis_corner_finding` | `analysis` | `(analysis_id, corner_id)` | Depende de `corner`. Pérdida **y** ganancia estimada por curva. Es la consulta por curva |
| `driver_index_dimension` | `driver_index` | `(driver_index_id, name)` | Conjunto abierto de dimensiones |
| `analysis_report_section` | `analysis_report` | `(report_id, key)` | Texto largo, ordenado |
| `analysis_report_assessment` | `analysis_report` | — | Fortalezas y mejoras. Depende de `corner`, opcional |
| `analysis_report_area_evaluation` | `analysis_report` | `(report_id, area_name)` | Nota opcional |
| `analysis_report_recommendation` | `analysis_report` | — | Depende de `corner`, opcional |
| `analysis_report_session` | puente | `(report_id, session_id)` | N ↔ M |
| `track_profile_corner` | `track_profile` | `(profile_id, corner_id)` | Clasificación fuerte o débil |

Las cinco tablas padre llevan `engine_version`, `computed_at` e `inputs_hash`, tal
como exige el modelo.

`analysis` lleva columna JSON para los errores detectados y para las ganancias
estimadas que **no** se atribuyen a una curva concreta, como el "0,2–0,3 s
mediante mejor rotación en curvas lentas" del informe de sesión, donde el sujeto
es una categoría y no una curva. `career` lleva en JSON sus fortalezas
consolidadas y sus debilidades persistentes.

La unicidad de `analysis_corner_finding` se establece por índice único parcial
sobre las filas que tienen curva, de modo que la fusión no pierde la garantía de
un solo hallazgo por análisis y curva.

**No existe un puente entre informe y curva.** Las curvas que un informe señala se
alcanzan por `analysis_report_assessment` y `analysis_report_recommendation`, que
ya apuntan a `corner`. Un puente añadiría un índice de información que ya existe un
nivel más abajo, y con datos que se regeneran periódicamente introduciría una
forma de incoherencia silenciosa: el puente podría afirmar que el informe menciona
una curva que no aparece en ninguna de sus hijas. La relación N↔M del modelo se
mantiene; se deriva en lugar de almacenarse, igual que el alcance de
`telemetry_file`.

**Lo que deliberadamente no se materializa.** `Career.pace_history`,
`metric_history` e `index_history` son series temporales sobre **sesiones**, no
sobre vueltas: recorrer 5.000 filas indexadas por fecha es trabajo trivial, así que
copiarlas sería duplicar datos sin ganancia. `career` guarda escalares, totales y
conclusiones consolidadas; las series se consultan en vivo.

El criterio, que vale para todo el sistema: **se materializa lo que agrega sobre
vueltas, no lo que agrega sobre sesiones.** Hay cien veces más vueltas que
sesiones.

**Dependencias.** Los derivados dependen de hechos, de catálogos y entre sí, en
este orden: `analysis → driver_index → analysis_report`, y `track_profile` y
`career` al final. El grafo no tiene ciclos.

## 5.5 Tablas técnicas

| Tabla | Contenido | Notas |
|-------|-----------|-------|
| `schema_migration` | Una fila por migración aplicada, con versión, nombre y fecha | Historial, no solo número actual |
| `app_setting` | Pares clave-valor: piloto activo, tema, rutas, preferencias | Es el destino del estado de interfaz que la auditoría del modelo expulsó de `Driver` |
| `import_batch` | Una fila por importación: fuente, fecha, resultado y registro original | Ver D3 |

Ninguna representa una entidad del dominio, y por eso no aparecen en
`DATA_MODEL.md`.

## 5.6 Grafo de dependencias

```
NIVEL 0   simulator    track    car    app_setting    schema_migration
             │           │       │
NIVEL 1   series      corner     │     import_batch
             │           │       │           │
NIVEL 2   season        │        │           │
             │          │        │           │
NIVEL 3   driver ───────┼────────┼───────────┘
             │          │        │
NIVEL 4   session ──────┴────────┘
             │
NIVEL 5   lap     telemetry_file     milestone
             │
NIVEL 6   analysis ──> driver_index ──> analysis_report
                                              │
NIVEL 7   objective ──────────────────────────┘
             │
NIVEL 8   objective_progress    track_profile    career
```

Dos propiedades del grafo que conviene no perder al implementar:

**No hay ciclos.** Una única arista sube de nivel permanente a nivel derivado, la
de `objective` hacia `analysis_report`, y es la que fuerza que `objective` se cree
después de los informes aunque sea una tabla permanente.

**El corte entre el nivel 5 y el 6 es limpio.** Todo lo que está por encima es
permanente; todo lo que está por debajo, salvo `objective`, es regenerable. Ese
corte es el que hace posible la estrategia de borrado de la sección 7 y la de
migraciones de la sección 8.

## 5.7 Simplificaciones aplicadas

El inventario partió de 45 tablas. Una revisión final buscó tablas justificadas
solo por normalización teórica y encontró cuatro. Se documentan para que nadie las
vuelva a añadir creyendo que faltan.

| Tabla suprimida | Destino | Motivo |
|-----------------|---------|--------|
| `analysis_report_corner` | Se deriva de las hijas del informe | No almacenaba nada nuevo, y al regenerarse podía contradecir a sus propias hijas |
| `analysis_estimated_gain` | Fusionada en `analysis_corner_finding` | Mismo grano que la pérdida por curva. Dos tablas para un hecho por análisis y curva |
| `career_assessment` | Columna JSON de `career` | Siete filas, sin claves ajenas, leídas siempre con su padre. Incumplía el criterio del apartado 4.1 |
| `session_flag` | Columna JSON de `session` | Vocabulario abierto y corto. Además "Pole" era `start_position = 1` almacenado dos veces |

Ninguna de las cuatro cambia el modelo de dominio: las relaciones y los atributos
que describían siguen existiendo, unos derivados y otros en columna JSON.

Dos de los cuatro hallazgos no eran exceso de normalización sino incoherencias
propias del diseño: el criterio del apartado 4.1 estaba mal aplicado a las
valoraciones de carrera, y la bandera "Pole" incumplía la regla de no almacenar lo
que puede calcularse. Merece la pena recordarlo, porque el mismo criterio y la
misma regla son los que hay que aplicar ante cualquier tabla nueva.

---

# 6. Índices previstos

Los índices se derivan de consultas reales, no de intuiciones. Cada uno de los
siguientes existe porque hay una pantalla o un cálculo que lo necesita.

## 6.1 Dos hechos de SQLite que condicionan todo el apartado

**SQLite no indexa automáticamente las columnas de clave ajena.** Indexa la clave
del padre, porque es primaria o única, pero no la del hijo. Sin un índice
explícito, buscar las vueltas de una sesión o propagar un borrado en cascada
recorre la tabla entera. Con 500.000 vueltas eso es la diferencia entre un
milisegundo y varios segundos.

**Un índice compuesto sirve para todos sus prefijos.** Un índice sobre
`(driver_id, track_id, car_id, started_at)` responde también a consultas por
`(driver_id, track_id)` y por `driver_id` solo. Es la razón por la que el
inventario de abajo es corto: un índice bien ordenado sustituye a tres.

## 6.2 Índices por consulta real

### Evolución del piloto

Serie de iRating o Safety Rating a lo largo de diez años.

| Índice | Papel |
|--------|-------|
| `session(driver_id, started_at)` | Filtra por piloto y devuelve ya ordenado: no hay que ordenar nada |
| `session_metric(session_id, name)` | Ya existe como restricción de unicidad. Resuelve la métrica de cada sesión |

Recorre unas 5.000 filas, no 500.000, porque la serie es por sesión. Es la razón
por la que estas series **no** se materializan.

### Sesiones por circuito

| Índice | Papel |
|--------|-------|
| `session(driver_id, track_id, car_id, started_at)` | Cubre las tres granularidades por la regla del prefijo, y entrega el orden |

Un solo índice para "mis sesiones", "mis sesiones en Bathurst" y "mis sesiones en
Bathurst con el GR86".

### Mejores vueltas

| Índice | Papel |
|--------|-------|
| `lap(session_id, lap_time_ms)` **parcial, solo vueltas válidas** | Mejor vuelta de una sesión sin leer las inválidas |
| `track_profile(driver_id, track_id, car_id)` | Récord del combo en una sola lectura, ya materializado |

El récord por combinación **no se calcula recorriendo vueltas**: se lee de
`track_profile`. La consulta cruzando `session` y `lap` existe como respaldo para
recalcular, no para pintar la interfaz.

El índice parcial es una capacidad de SQLite que conviene aprovechar: indexar solo
las vueltas válidas reduce su tamaño y evita filtrar después.

### Análisis por curva

| Índice | Papel |
|--------|-------|
| `analysis_corner_finding(corner_id, analysis_id)` | "Cuánto pierdo históricamente en Forrest's Elbow" |
| `analysis_corner_finding(analysis_id, corner_id)` | Ya existe como unicidad parcial. "Dónde perdí tiempo en esta sesión" |
| `corner(track_id, name)` · `corner_alias(track_id, alias)` | Resolver el nombre y sus variantes |

Las dos direcciones son necesarias. Es la lección de toda relación N↔M: un índice
por sentido, porque se consulta por los dos.

### Comparación entre temporadas

| Índice | Papel |
|--------|-------|
| `session(season_id, started_at)` | Agrupa las sesiones de cada temporada |
| `season(series_id, year, season_number)` | Ya existe como unicidad. Localiza la temporada |

### Búsqueda de telemetrías

| Índice | Papel |
|--------|-------|
| `telemetry_file(content_hash)` | Único. Camino caliente de la importación: decide en una lectura si el fichero ya entró |
| `telemetry_file(session_id)` | Ficheros de una sesión, y cascada de borrado |
| `telemetry_file(lap_id)` | Ficheros de una vuelta concreta, caso Garage61 |

## 6.3 Índices de servicio

| Índice | Para qué |
|--------|----------|
| `lap(session_id)` | Cubierto por el índice parcial anterior solo para vueltas válidas; hace falta el completo para la cascada de borrado |
| `analysis(engine_version)` | Encontrar lo que quedó obsoleto tras actualizar el Core |
| `analysis_report_session(session_id, report_id)` | Sentido inverso del puente: "informes que cubren esta sesión" |
| `objective(driver_id, priority_order)` | Lista de objetivos del piloto, ya ordenada |
| `objective_progress(status)` | Objetivos activos para el dashboard |
| `milestone(driver_id, achieved_on)` | Cronología de logros |
| `driver_index(driver_id, computed_at)` | Ya existe como unicidad. Última fotografía y evolución |
| `session(import_batch_id)` | Revertir o auditar una importación |
| Los 7 `(source, external_id)` | Únicos. Deduplicación en la importación |

## 6.4 Presupuesto y disciplina

Alrededor de **30 índices** en total, contando los que ya vienen dados por las
restricciones de unicidad. Reglas para que no crezcan sin control:

1. Un índice nuevo se justifica con la consulta concreta que lo necesita.
2. Antes de crear uno, comprobar si un índice existente lo cubre por prefijo.
3. No indexar columnas de baja cardinalidad en solitario; solo como parte de un
   compuesto.
4. Cada índice se paga en cada escritura. En la importación por lotes eso se nota.
5. Ejecutar la recogida de estadísticas después de importaciones grandes: sin
   estadísticas, el planificador elige mal aunque el índice exista.

---

# 7. Estrategia de borrado

## 7.1 Qué nunca debe borrarse

| Dato | Protección técnica |
|------|--------------------|
| `session`, `lap`, `session_metric` | Sin operación de borrado expuesta salvo la eliminación explícita de una sesión por el usuario |
| `milestone` y su certificado en disco | La sesión que lo originó queda **protegida contra borrado** mientras el hito exista |
| `telemetry_file` (la fila) | Sobrevive a la desaparición del fichero. La fila registra que existió y con qué huella |
| `objective` (definición) | Cumplido, abandonado o superado, permanece. Es la historia de las intenciones |
| `driver` | Solo desaparece en un reinicio completo de perfil, con confirmación explícita |
| `import_batch` (la fila) | Su registro original se puede archivar; la fila, no |

## 7.2 Qué puede recalcularse

Se actualiza en el sitio, conservando identidad.

| Dato | Disparador |
|------|-----------|
| `analysis_report` y sus hijas | Nueva sesión en el periodo, o cambio en el motor de coaching |
| `objective_progress` | Cada sesión nueva |

`analysis_report` se recalcula y **nunca se recrea**, por las dos razones del
apartado 2.4: es el único destino de una clave ajena desde una tabla permanente, y
es un documento que el piloto leyó.

## 7.3 Qué puede regenerarse

Se puede vaciar la tabla completa y reconstruirla desde cero.

| Dato | Reconstruible desde | Coste estimado |
|------|--------------------|----------------|
| `analysis` y sus hijas | `session` + `telemetry_file` | Alto: hay que releer telemetría |
| `driver_index` y sus dimensiones | `analysis` | Bajo |
| `track_profile` y sus hijas | Sesiones del combo | Bajo |
| `career` y sus hijas | Todo el histórico | Bajo |
| `objective_progress` | `objective` + sesiones | Trivial |

La regeneración de `analysis` es la única costosa, porque exige volver a leer los
ficheros de telemetría. De ahí que el estado de disponibilidad de esos ficheros
(apartado 7.4) sea información crítica: sin fichero no hay análisis reconstruible,
solo el resultado ya calculado.

## 7.4 Qué puede archivarse

Aquí está el único dato realmente voluminoso del sistema.

**Los ficheros de telemetría en disco.** Decenas de gigabytes tras diez años. La
fila de `telemetry_file` permanece siempre; el fichero puede comprimirse, moverse a
otro volumen o eliminarse.

Esto exige una columna técnica de disponibilidad, permitida por la frontera del
apartado 1 porque describe infraestructura y no dominio:

| Estado | Significado |
|--------|-------------|
| `available` | El fichero está donde dice la ruta y su huella coincide |
| `archived` | Retirado a propósito. Se sabe dónde y se puede recuperar |
| `missing` | No está y no se retiró a propósito. Requiere aviso al usuario |

**El registro original de importación.** `import_batch` puede vaciar su carga
pasado un tiempo, conservando la fila con la fecha, la fuente y el resultado.

La política concreta —cuántas temporadas se conservan sin archivar, si se comprime
o se mueve— es la decisión pendiente 4.

## 7.5 Qué puede eliminar el usuario

Cuatro operaciones, y ninguna más.

| Operación | Alcance | Efecto |
|-----------|---------|--------|
| Eliminar una sesión importada por error | La sesión y todo lo que cuelga de ella | Cascada a vueltas, métricas, banderas, ficheros y análisis |
| Eliminar un objetivo | El objetivo y su progreso | Los objetivos que lo refinan quedan sin antecesor |
| Eliminar un fichero de telemetría | El fichero en disco | La fila pasa a `archived`. El análisis ya calculado sobrevive |
| Reiniciar el perfil | Todo lo del piloto | Los catálogos permanecen |

Tres reglas que gobiernan las cuatro:

**El borrado ocurre solo en la raíz.** Se elimina una sesión completa, nunca una
vuelta suelta. Una vuelta de una carrera ocurrió, y borrarla falsificaría el
historial. La misma lógica vale para las métricas y las banderas.

**El borrado se rechaza si destruiría un hecho certificado.** Si una sesión
originó un hito, la operación se rechaza con una explicación, en lugar de perder el
certificado o dejarlo huérfano.

**El borrado es una transacción y deja margen de rescate.** Todo ocurre o nada
ocurre. Y el fichero de telemetría en disco no se destruye en el mismo acto: pasa a
una cola de purga, de modo que un borrado accidental se recupera reimportando.

Tras un borrado grande conviene compactar el fichero de base de datos: SQLite no
devuelve al sistema el espacio liberado por su cuenta salvo que se configure el
vaciado incremental.

## 7.6 Matriz de comportamiento en cascada

Ninguna clave ajena se deja con el comportamiento por omisión. Cada una se decide.

| Relación | Al borrar el padre | Motivo |
|----------|--------------------|--------|
| Hija dentro del mismo agregado (`lap`, `session_metric`, hijas de informe, dimensiones, hallazgos por curva) | **CASCADE** | No tienen vida propia fuera de su padre |
| `session` → `driver` | **RESTRICT** | Un piloto con historial no se borra por accidente |
| `session` → `track`, `car`, `season`, `simulator` | **RESTRICT** | Protege los catálogos en uso |
| `milestone` → `session` | **RESTRICT** | Protege el logro certificado |
| `analysis` → `session` | **CASCADE** | Derivado de la sesión |
| `analysis.reference_lap_id` → `lap` | **SET NULL** | Es una referencia de comparación, no una dependencia |
| `track_profile.best_lap_id` → `lap` | **SET NULL** | Es una caché |
| `objective` → `analysis_report` | **SET NULL** | El objetivo sobrevive y solo pierde procedencia |
| `objective` → `milestone` | **RESTRICT** | Coherente con proteger el hito |
| `objective_progress` → `objective` | **CASCADE** | Es su parte derivada |
| Puente `analysis_report_session` | **CASCADE** por ambos lados | Un puente sin extremos no significa nada |
| Tablas `*_external_id` | **CASCADE** | Identidad externa de una fila que ya no existe |

---

# 8. Versionado y migraciones

El objetivo es evolucionar durante años sin romper instalaciones. En una
aplicación de escritorio esto es más delicado que en un servidor: no hay una sola
base de datos que actualizar, sino una por usuario, cada una en una versión
distinta y sin nadie vigilando.

## 8.1 Mecanismo

| Pieza | Papel |
|-------|-------|
| Versión de esquema del propio fichero | Número de versión que viaja dentro del fichero de base de datos |
| Tabla `schema_migration` | Historial: una fila por migración aplicada, con nombre y fecha |

Se usan las dos. El número dentro del fichero permite decidir en un instante si
hay que migrar; la tabla conserva el historial, que es lo que permite diagnosticar
una instalación rara años después.

## 8.2 Seis reglas

1. **Solo hacia adelante.** No hay migraciones de reversión. Para volver atrás se
   restaura la copia previa.
2. **Numeradas y secuenciales.** Sin huecos ni ramas.
3. **Inmutables una vez publicadas.** Una migración que ya se ejecutó en la
   máquina de alguien no se edita nunca. Si estaba mal, se corrige con una nueva.
4. **Una migración, una transacción.** Si falla a medias, la base de datos queda
   como estaba.
5. **Copia de seguridad automática antes de migrar.** La base de datos es un solo
   fichero, así que copiarla es trivial y es la red de seguridad más barata que
   existe.
6. **Negarse a abrir una base más nueva que la aplicación.** Si el usuario abre con
   una versión antigua un fichero ya migrado, la aplicación avisa en lugar de
   corromperlo.

## 8.3 Lo que SQLite permite y lo que no

Esto condiciona el diseño más de lo que parece.

| Operación | Coste |
|-----------|-------|
| Añadir una tabla | Trivial |
| Añadir una columna | Trivial |
| Añadir un índice | Trivial |
| Renombrar tabla o columna | Directo en versiones modernas |
| Eliminar una columna | Directo en versiones modernas, con salvedades si está indexada |
| **Cambiar un tipo, una restricción o añadir una clave ajena** | **Reconstrucción completa de la tabla** |

La última fila es la que importa: **una clave ajena no se puede añadir a una tabla
que ya existe** sin recrearla, copiar los datos y rehacer sus índices. En una tabla
de 500.000 vueltas eso es una operación larga y delicada en el equipo del usuario.

Consecuencia directa: **las columnas de clave ajena tienen que estar bien desde la
primera versión.** Es la razón por la que el orden de implementación de la sección
11 sigue el grafo de dependencias en lugar de la clasificación por categorías.

## 8.4 Tipos de migración y su coste

| Tipo | Ejemplo realista | Coste |
|------|------------------|-------|
| Tabla nueva | Llegan `Setup`, `Sector` o `Team` con Garage61 | Trivial |
| Columna nueva | Un dato de `attributes` se promociona a columna propia | Trivial |
| Índice nuevo | Una pantalla nueva necesita otro orden | Trivial |
| **Reestructurar derivados** | `analysis` gana una tabla hija | **Cero migración de datos**: se tira y se recalcula |
| Reestructurar permanentes | Cambiar el tipo de una columna de `lap` | Alto: reconstrucción y copia previa |

**La cuarta fila es el mayor beneficio práctico de la separación entre hechos y
derivados.** La mayoría de la evolución de DDP durante los próximos años ocurrirá
en la capa de análisis, informes e índices, es decir, en tablas regenerables. Esas
migraciones no mueven datos: vacían y recalculan. Una instalación con diez años de
historial las supera sin riesgo, porque los hechos no se tocan.

## 8.5 Versión del motor, distinta de la versión del esquema

Son dos ejes independientes y conviene no confundirlos.

| Eje | Qué cambia | Cómo se resuelve |
|-----|-----------|------------------|
| Versión de esquema | La forma de las tablas | Migración |
| `engine_version` en filas derivadas | La fórmula que las calculó | Recálculo selectivo |

Mejorar el cálculo del Driver Index no es una migración: es marcar como obsoletas
las filas con `engine_version` anterior y recalcularlas en segundo plano. El índice
sobre `analysis(engine_version)` existe exactamente para eso.

## 8.6 Cómo se prueba

Se conserva un fichero de base de datos de ejemplo por cada versión histórica
publicada. Antes de cada entrega, todas se migran a la última versión y se
verifica la integridad. Es la única forma honesta de saber que una instalación de
hace cinco años sigue funcionando.

---

# 9. Rendimiento a diez años

## 9.1 El escenario

| Magnitud | Valor |
|----------|-------|
| Historial | 10 años |
| Sesiones | más de 5.000 |
| Vueltas | más de 500.000 |
| Telemetría | decenas de gigabytes |

## 9.2 Tamaño estimado de la base de datos

| Tabla | Filas | Datos + índices |
|-------|------:|----------------:|
| `lap` | 500.000 | ~60 MB |
| `analysis_report_section` | ~33.000 | ~28 MB, dominado por texto |
| `analysis_corner_finding` | ~50.000 | ~5 MB |
| `driver_index_dimension` | ~50.000 | ~4 MB |
| `session` y sus hijas | ~20.000 | ~4 MB |
| `telemetry_file` | 5.000+ | ~3 MB |
| `lap_external_id` | pocos miles | ~2 MB |
| `analysis`, `driver_index`, `analysis_report` | ~15.000 | ~7 MB |
| Catálogos, usuario, perfiles, carrera | pocos miles | ~5 MB |
| Identificadores UUID de las fundamentales | — | ~20 MB |
| **Total** | | **~140 MB** |
| `import_batch` con carga original | 5.000 | archivable, hasta ~250 MB |

**La conclusión que importa:** unos 140 MB después de diez años. SQLite trabaja con
comodidad en órdenes de magnitud superiores. Las decenas de gigabytes de telemetría
no están aquí porque nunca entran en la base de datos.

La estimación anterior era de unos 180 MB y sobreestimaba `lap_external_id` al
suponerla paralela a `lap`. Al ser dispersa, la tabla baja de unos 45 MB a unos 2, y
es de largo la corrección más grande del apartado.

Es decir, el problema de escala de DDP no es el tamaño de la base de datos, sino
elegir bien qué se consulta. Los apartados siguientes van de eso.

## 9.3 Presupuesto de respuesta

| Pantalla u operación | Objetivo | Cómo se consigue |
|----------------------|---------:|------------------|
| Dashboard completo | < 50 ms | Lee de `career` y `track_profile`, ya agregados. No toca `lap` |
| Página de lista de sesiones | < 30 ms | `session(driver_id, started_at)`, 50 filas, orden ya dado |
| Detalle de una sesión | < 80 ms | Una sesión, sus vueltas por índice y su análisis 1:1 |
| Gráfico de evolución a 10 años | < 150 ms | ~5.000 filas de sesión, no 500.000 de vuelta |
| Análisis histórico de una curva | < 100 ms | `analysis_corner_finding(corner_id, ...)` |
| Importar una sesión | < 500 ms | Una transacción; el coste real es leer el fichero |
| Recalcular toda la capa derivada | Minutos, en segundo plano | Por lotes, sin bloquear la lectura |

## 9.4 Ocho principios

**1. La telemetría nunca entra en la base de datos.** Es la decisión que mantiene
el sistema en 140 MB en lugar de decenas de gigabytes.

**2. Las filas calientes son estrechas.** El texto largo de los informes vive en
`analysis_report_section`, y las cargas de importación en `import_batch`. Las
tablas que se recorren se mantienen delgadas, porque el coste de un recorrido es
proporcional a los bytes leídos, no a las filas.

**3. Se materializa lo que agrega sobre vueltas; no lo que agrega sobre sesiones.**
Hay cien veces más vueltas que sesiones. `track_profile` y `career` existen para
que el dashboard no recorra 500.000 filas; las series por sesión se consultan en
vivo porque son 5.000.

**4. Toda clave ajena que se use para buscar o para cascadear lleva índice.**
SQLite no los crea solos, y es el error de rendimiento más común y más silencioso.

**5. Ninguna lista se devuelve entera.** Paginación siempre, y por clave ordenada
en lugar de por desplazamiento: saltarse 100.000 filas cuesta recorrerlas.

**6. La interfaz lee de la capa derivada; solo el recálculo lee los hechos.** Si
una pantalla necesita agregar hechos en vivo, o le falta un índice o le falta un
derivado.

**7. Importaciones y recálculos por lotes, en transacciones agrupadas.** Confirmar
cada fila por separado es el otro error clásico: multiplica por cien el tiempo de
importación.

**8. Escritura en modo diario adelantado.** Permite que el recálculo en segundo
plano escriba mientras la interfaz lee, sin bloqueos. Es imprescindible para que
recalcular diez años no congele la aplicación.

## 9.5 Configuración del motor

| Ajuste | Valor previsto | Cuándo se aplica |
|--------|----------------|------------------|
| Modo de diario | Adelantado (WAL) | Una vez; queda en el fichero |
| Claves ajenas | Activadas | **En cada conexión**, sin excepción |
| Sincronización | Normal | En cada conexión; seguro combinado con WAL |
| Espera por bloqueo | ~5 s | En cada conexión |
| Caché de páginas | ~64 MB | En cada conexión |
| Tablas temporales | En memoria | En cada conexión |
| Proyección en memoria | ~256 MB | En cada conexión |
| Tamaño de página | 4096 u 8192 bytes | **Al crear la base.** Cambiarlo después exige compactar el fichero entero |
| Vaciado incremental | Activado | **Al crear la base.** Ídem |
| Estadísticas del planificador | Recogida periódica | Tras importaciones grandes |

Dos avisos que ahorran depuraciones largas: la activación de claves ajenas viene
**desactivada por omisión y es por conexión**, así que debe aplicarse en el único
punto donde se crean conexiones y nunca consulta a consulta; y el tamaño de página
y el modo de vaciado conviene fijarlos al crear la base, porque cambiarlos más
tarde obliga a reescribir el fichero completo en el equipo del usuario.

---

# 10. Integridad

Cuatro problemas a evitar y cómo se evita cada uno.

## 10.1 Datos huérfanos

| Medida | Detalle |
|--------|---------|
| Claves ajenas activadas siempre | En el punto único de creación de conexiones. Es la causa número uno de huérfanos en SQLite |
| Cero referencias polimórficas | Decisiones D4 y D6. No hay ni una columna cuyo destino dependa de otra columna |
| Comportamiento explícito en cada clave ajena | La matriz del apartado 7.6. Ninguna se queda con el valor por omisión |
| Índice en cada clave ajena hija | Una cascada sin índice es tan lenta que invita a desactivarla, y ahí empiezan los huérfanos |

Con estas cuatro medidas, un huérfano es **imposible** dentro de la base de datos.
Fuera de ella no, y de eso trata el apartado 10.4.

## 10.2 Duplicados

La restricción es la última línea de defensa, no la primera. Lo que de verdad evita
duplicados es un procedimiento de reconocimiento determinista en la importación:

```
1. ¿Coincide (source, external_id)?      -> es la misma fila. Actualizar
2. ¿Coincide la clave natural?           -> es la misma fila. Vincular el id externo
3. ¿Coincide un alias conocido?          -> es la misma fila. Vincular el alias
4. Ninguna coincidencia                  -> crear fila nueva
```

Respaldado por restricciones de unicidad:

| Unicidad | Impide |
|----------|--------|
| Las 7 de `(source, external_id)` | Importar dos veces la misma subsesión, coche o vuelta |
| `telemetry_file.content_hash` | Importar dos veces el mismo fichero, aunque se renombre |
| `session(driver_id, started_at)` | Dos sesiones del mismo piloto a la misma hora |
| `lap(session_id, lap_number)` | Dos vueltas con el mismo número |
| `corner(track_id, name)` | Dos curvas con el mismo nombre en un circuito |
| `track_alias.alias`, `car_alias.alias` | Que "Bathurst" apunte a dos circuitos distintos |
| `season(series_id, year, season_number)` | Temporadas repetidas |
| `session_metric(session_id, name)` | Dos valores de iRating para la misma sesión |
| `track_profile(driver_id, track_id, car_id)` | Dos perfiles del mismo combo |
| `career.driver_id` | Dos carreras para un piloto |

## 10.3 Inconsistencias

**Restricciones de valor cerrado, solo donde el vocabulario es cerrado.** Aquí hay
que tener cuidado, porque el modelo garantiza extensibilidad precisamente dejando
varios vocabularios abiertos. Restringirlos rompería esa garantía.

| Cerrado, se restringe | Abierto, **nunca** se restringe |
|-----------------------|--------------------------------|
| `driver.kind` | `session.session_type` |
| `analysis_report.scope` | `area_evaluation.area_name` |
| `objective.status` y `objective.nature` | `corner.category` |
| `assessment.kind` y `subject_type` | Formato de telemetría |
| `telemetry_file.availability` | Nombre de dimensión del índice |
| `analysis.reference_source` | Nombre de métrica de sesión |
| | Clave de sección de informe |

**Restricciones de rango.** Tiempos de vuelta positivos, porcentajes entre 0 y 100,
posiciones positivas, incidencias no negativas, notas dentro de su escala.

**Unidades por convención de nombre.** Toda columna en milisegundos termina en
`_ms`. Es una medida barata y sorprendentemente eficaz contra el error de mezclar
segundos y milisegundos, que es el fallo más probable de todo el proyecto dada la
cantidad de tiempos que maneja.

**Marcas de tiempo en texto ISO-8601 en UTC**, como fija el modelo. Ordenan
correctamente por comparación de texto, se leen sin descifrar y no dependen de la
zona horaria del usuario.

**Una fila derivada por padre y versión de motor.** Impide que convivan en
silencio resultados de dos versiones del cálculo mezclados en la misma pantalla.

## 10.4 Referencias rotas

Hay dos clases de invariante que el motor no puede vigilar. Conviene nombrarlas en
lugar de suponer que las restricciones lo cubren todo.

**Los ficheros en disco.** Es la única referencia que sale de la base de datos, y
justamente la que apunta a las decenas de gigabytes. Se protege con la huella de
contenido, el estado de disponibilidad del apartado 7.4 y una revisión periódica
que compare filas con ficheros y marque los que faltan.

**Invariantes de aplicación.** Se comprueban en el Core, no en el esquema:

| Invariante | Por qué no lo cubre el motor |
|------------|------------------------------|
| La cadena `objective.supersedes_id` debe ser acíclica | Requiere recorrer la cadena |
| Un informe debe cubrir al menos una sesión | Requiere contar filas del puente |
| Las dimensiones de un índice deben pertenecer a su fotografía | Se cubre por clave ajena, pero su coherencia semántica no |

Un caso que **sí** se puede dejar al motor y merece la pena: "como máximo una
vuelta marcada como la mejor de su sesión" se resuelve con un índice único parcial,
sin código.

## 10.5 Rutina de verificación

Se ejecuta a petición del usuario y **obligatoriamente después de cada migración**:

1. Comprobación de integridad del propio fichero.
2. Búsqueda de huérfanos, que debe salir vacía si las claves ajenas están activas.
3. Cotejo de ficheros de telemetría en disco frente a sus filas.
4. Búsqueda de duplicados por clave natural.
5. Recuento de filas derivadas con versión de motor obsoleta.
6. Comprobación de aciclicidad de las cadenas de objetivos.

El resultado se presenta al usuario en lenguaje llano, no como un volcado técnico.

---

# 11. Decisiones pendientes

Cuatro. Ninguna bloquea la fase 0 ni la 1.

**1. Forma del identificador opaco.** Recomendación del apartado 4.2: clave
primaria entera para los cruces y UUID en BLOB de 16 bytes en las 12 entidades
fundamentales. La alternativa de prescindir del UUID ahorra unos 20 MB y cierra la
puerta a fusionar instalaciones más adelante.

**2. Granularidad del registro original de importación.** Recomendación del
apartado 4.4: uno por lote de importación en lugar de uno por fila, lo que evita
unos 250 MB de texto que casi nunca se lee.

**3. Alcance del borrado de sesiones por el usuario.** Si se permite el borrado
real con cascada, o solo archivar la sesión manteniendo la fila. Afecta a la
interfaz más que al esquema, pero conviene decidirlo antes de la fase 3.

**4. Política de archivado de telemetría.** Cuántas temporadas se conservan
accesibles, y si lo archivado se comprime o se traslada. Necesaria antes de que el
volumen crezca, no antes de empezar.

---

# 12. Orden de implementación

El orden sigue el grafo de dependencias del apartado 5.6, no la clasificación por
categorías. El motivo es el del apartado 8.3: añadir una clave ajena a una tabla
que ya existe obliga a reconstruirla, así que cada tabla debe nacer con sus claves
ajenas completas.

SQLite tolera que una tabla mencione a otra que aún no existe, así que el orden no
es una obligación técnica del motor. Es una obligación de método: permite entregar
por fases, y sobre todo permite **verificar cada fase con datos reales** antes de
seguir.

Cada fase se valida reproduciendo evidencia concreta del apéndice de
`DATA_MODEL.md`. Si la fase no reproduce su dato, no está terminada.

| # | Fase | Tablas | Verificación |
|:-:|------|--------|--------------|
| **0** | Infraestructura | `schema_migration`, `app_setting`, configuración del motor, punto único de conexión, copia previa a migrar | Una migración se aplica, se registra y la copia se crea |
| **1** | Catálogos | `simulator`, `car`, `track`, `corner`, `series`, `season` y sus alias, identificadores externos y calendario | Cargar Mount Panorama con sus 9 curvas, el Toyota GR86 y la iRacing Sports Car Series. "Bathurst" debe resolver al circuito |
| **2** | Piloto | `driver`, hardware, `driver_external_id` | Cargar el perfil de [P] con su pedalera y volante |
| **3** | Importación y sesión | `import_batch`, `session`, `session_metric`, `session_external_id`, `lap`, `lap_external_id`, `telemetry_file` | Cargar las 4 sesiones de [M] y las 2 carreras de [W]. Reimportar no debe duplicar nada. El iRating debe existir en un solo sitio y "Pole" no debe almacenarse |
| **4** | Análisis | `analysis`, `analysis_corner_finding` | Vaciar y reconstruir debe dar el mismo resultado. Pérdida y ganancia por curva deben leerse juntas |
| **5** | Índice del piloto | `driver_index`, `driver_index_dimension` | Reproducir el 89,3 de [P] con 9 dimensiones y el 90,8 de [I] con 6, sin cambiar el esquema |
| **6** | Informes | `analysis_report`, sus 4 hijas y el puente con sesiones | Reproducir el Informe 001 de [I] y el Semanal 001 de [W], incluida el área sin nota. Las curvas señaladas deben obtenerse de las hijas, sin puente propio |
| **7** | Intención | `milestone`, `objective`, `objective_progress` | La cadena "Bajar de 2:30" → "de forma consistente" → "2:29.2-2:29.4", y "Ganar una carrera" cerrado por "First Win" |
| **8** | Estado acumulado | `track_profile`, `track_profile_corner`, `career` | `track_profile` debe reproducir la ficha de [C]: récord 2:29.907, 4 curvas fuertes y 3 a mejorar. `career` debe devolver las 4 fortalezas y 3 debilidades de [P] desde su columna JSON |
| **9** | Índices y ajuste | Los ~30 índices y la recogida de estadísticas | Medir contra el presupuesto del apartado 9.3 con datos sintéticos de 500.000 vueltas |
| **10** | Verificación | La rutina del apartado 10.5 | Debe salir limpia sobre la base cargada en las fases anteriores |

**Por qué la fase 7 va después de la 6**, aunque `objective` y `milestone` sean
datos permanentes y los informes sean derivados: `objective` tiene una clave ajena
hacia `analysis_report`, la única del esquema que sube de permanente a derivado, y
esa columna debe existir desde el nacimiento de la tabla.

**Por qué la fase 9 va al final.** Los índices se miden, no se adivinan. Crearlos
antes de tener volumen es diseñar a ciegas, y además ralentiza la carga de las
fases anteriores. La fase 9 incluye generar medio millón de vueltas sintéticas
precisamente para poder comprobar los objetivos de respuesta.

---

# 13. Cierre

## Estado del documento

| Aspecto | Situación |
|---------|-----------|
| Base | `DATA_MODEL.md` v1.0, sin modificar |
| Tablas diseñadas | 41: 14 catálogo, 5 usuario, 6 sesión, 13 derivadas, 3 técnicas |
| Entidades futuras | 3, sin tabla, por decisión explícita |
| Índices previstos | ~30, cada uno con su consulta |
| Decisiones pendientes | 4, ninguna bloqueante |
| SQL escrito | Ninguno |

## Reglas de uso

1. La implementación de SQLite se deriva de este documento, y este documento se
   deriva de `DATA_MODEL.md`. La cadena no se recorre en sentido contrario.
2. Una tabla nueva que no corresponda a una entidad del modelo solo puede ser
   técnica, en el sentido del apartado 1.
3. Si al implementar hace falta un atributo de dominio nuevo, se detiene el trabajo
   y se abre la versión 1.1 del modelo. No se improvisa una columna.
4. Ninguna clave ajena queda sin comportamiento de borrado decidido, y ninguna
   clave ajena hija queda sin índice.
5. Los vocabularios que el modelo deja abiertos no se restringen nunca en el
   esquema.
6. Ninguna fase se da por terminada sin reproducir su dato de verificación.
7. **Ninguna modificación estructural durante la implementación.** Ni una tabla, ni
   una columna, ni una clave ajena que no esté en este documento. Ante una necesidad
   estructural el desarrollo se detiene y se abre la versión 1.1.

**Versión 1.0 · congelada · referencia oficial para la implementación de SQLite.**


