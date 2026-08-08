# DDP — Filosofía oficial

> Este es el documento más importante del proyecto.
> Es la **referencia absoluta** de producto.
>
> No describe APIs, motores ni implementación.
> Define **qué es DDP**, para qué existe y cómo debe decidirse
> cualquier desarrollo futuro.
>
> Está por encima de RFCs, motores y decisiones de implementación.
> Ante duda de producto, **este documento es el criterio de decisión**.
>
> Los RFCs, motores, módulos, pantallas y funcionalidades deben
> respetarlo. La documentación técnica explica *cómo* se construye;
> esta filosofía explica *por qué* y *hacia dónde*.

---

# Nuestra misión

DDP no persigue que hagas tu mejor vuelta.

Persigue que te conviertas en el mejor piloto que puedes llegar a ser.

---

## ¿Qué es realmente DDP?

DDP es un **sistema de desarrollo de pilotos**.

No es un analizador de telemetrías. La telemetría es la materia prima;
el producto es el **piloto que mejora**.

El objetivo nunca es únicamente mejorar una vuelta.
El objetivo es **desarrollar al piloto a largo plazo**.

DDP no toma decisiones pensando únicamente en la siguiente vuelta.

DDP toma decisiones pensando en el piloto que el usuario será
**dentro de seis meses**.

Cada recomendación, cada objetivo y cada plan deben poder leerse así:
¿esto construye al piloto del futuro, o solo maquilla el resultado de hoy?

### DDP no es

- un visor de telemetrías
- un analizador de sesiones
- un generador de informes
- un panel de estadísticas

Esas piezas pueden existir como **instrumentos**. Ninguna de ellas es
el destino del proyecto.

### DDP sí es

- un **entrenador personal**
- un **ingeniero de pista**
- un **sistema de desarrollo continuo**
- un **compañero** que conoce la evolución completa del piloto

### La telemetría

La telemetría no es el producto.

Es únicamente la fuente de información.

La telemetría solo existe para **comprender al piloto**.
El verdadero producto es la **evolución del piloto**.

Nunca debemos diseñar funcionalidades cuyo objetivo sea simplemente
mostrar más datos. Si la telemetría no se traduce en comprensión,
decisión o entrenamiento con propósito, se ha usado mal.

---

## Filosofía

Toda funcionalidad futura debe responder siempre a esta pregunta:

> ¿Esto ayuda al piloto a convertirse en un piloto más completo,
> más rápido y más inteligente?

Si la respuesta es no, probablemente no pertenece a DDP.

DDP no acumula datos por acumularlos.
No presume de gráficos.
No confunde actividad con progreso.

Mide el éxito por una sola cosa: **la evolución del piloto**.

### Evolución continua

DDP acompaña al piloto durante **toda su carrera deportiva**.

No analiza sesiones aisladas.
Construye una **historia**.

Aprende del piloto.
Recuerda decisiones anteriores.
Adapta el entrenamiento.
Acompaña el crecimiento.

Una tanda no cierra el juicio.
Abre el siguiente capítulo del mismo arco.

### Confianza

El objetivo final no es impresionar.
Es que el piloto **confíe en DDP**.

No porque sea una IA.
No porque tenga muchas métricas.

Sino porque siempre puede explicar:

- por qué recomienda algo
- por qué mantiene un objetivo
- por qué cambia el foco
- qué evidencia utiliza

Sin explicación, no hay confianza.
Sin confianza, no hay desarrollo real: solo otra pantalla que se ignora.

La confianza es un **pilar** del proyecto.
Todo lo que debilite la capacidad de justificar una decisión
debilita DDP.

---

## Principios

1. **La telemetría es un medio, nunca el objetivo.**
   Importamos vueltas para entender al piloto, no para coleccionar archivos.

2. **Cada sesión forma parte de una historia mucho mayor.**
   Una tanda no es un veredicto. Es un capítulo.

3. **El progreso importa más que una vuelta rápida.**
   Una mejora sostenida vale más que un pico aislado.

4. **Entrenar siempre es más importante que analizar.**
   El análisis existe para decidir cómo entrenar mejor.

5. **Todo entrenamiento debe tener un propósito claro.**
   Si no hay propósito, no hay programa: hay ruido.

6. **Todo objetivo debe tener una explicación.**
   El piloto merece saber *por qué* trabaja eso ahora.

7. **Toda recomendación debe poder justificarse.**
   Sin evidencia, no hay consejo: hay opinión. DDP no inventa.

8. **El Coach enseña, no solo analiza.**
   Señalar un problema no basta; debe orientar el siguiente paso.

9. **El sistema recuerda la evolución completa del piloto.**
   No reinicia la conversación en cada importación.

10. **El histórico tiene más valor que una sesión aislada.**
    El patrón en el tiempo importa más que el detalle de un día.

11. **El éxito se mide por la evolución del piloto, no por la cantidad de datos mostrados.**
    Más pantallas no equivalen a más desarrollo.

12. **La experiencia del piloto siempre tiene prioridad sobre la complejidad técnica.**
    Si el piloto no entiende, no actúa, o no confía, el sistema ha fallado
    aunque el motor sea brillante.

13. **La confianza se gana con evidencia y explicación, no con volumen.**
    El piloto debe poder preguntar “¿por qué?” y recibir una respuesta
    anclada a hechos, no a magia.

14. **Pensar en el piloto de dentro de seis meses.**
    Lo urgente de hoy no debe sabotear el desarrollo de mañana.

---

## Visión

No es un roadmap técnico. Es la dirección del proyecto.

**Hoy**

Analiza sesiones.
Convierte hechos de pista en evidencia usable.

**Después**

Interpreta la evolución.
Lee el arco del piloto, no solo la última tanda.

**Más adelante**

Construye planes de entrenamiento.
Transforma el diagnóstico en trabajo concreto con propósito.

**Después**

Actúa como entrenador durante la sesión.
Acompaña el aprendizaje mientras ocurre, no solo al revisar.

**Finalmente**

Se convierte en un ingeniero de pista inteligente capaz de interpretar
la situación completa del piloto durante entrenamientos, clasificación
y carrera.

El destino no es “más análisis”.
El destino es **un compañero de desarrollo** que conoce al piloto
y le ayuda a llegar más lejos de lo que llegaría solo.

---

## Cómo decidir si una funcionalidad pertenece a DDP

Antes de desarrollar cualquier característica, debe responderse:

1. ¿Ayuda realmente al piloto a mejorar?
2. ¿Hace al piloto más completo?
3. ¿Hace al piloto más rápido?
4. ¿Hace al piloto más inteligente?

Si la respuesta es no, esa funcionalidad **no debe implementarse**.

También debe preguntarse:

5. ¿Acerca al piloto que será dentro de seis meses?
6. ¿Puede explicarse con evidencia por qué existe?
7. ¿Aumenta la confianza del piloto en DDP?

Criterios de rechazo rápidos:

- Sirve solo para “ver más datos” sin cambiar decisiones de entrenamiento.
- Prioriza la complejidad del sistema sobre la claridad del piloto.
- Premia la sesión aislada y olvida la historia.
- Analiza sin enseñar, mide sin orientar, o recomienda sin justificar.
- No puede explicar por qué recomienda, mantiene o cambia un foco.

Criterios de aceptación rápidos:

- Acorta el camino entre evidencia y acción.
- Fortalece un hábito o habilidad con propósito.
- Hace más legible el progreso a lo largo del tiempo.
- Aumenta la confianza del piloto en *qué* trabajar y *por qué*.
- Sirve al desarrollo a largo plazo, no solo al resultado de hoy.

---

## Relación con el resto del proyecto

| Documento | Rol |
|-----------|-----|
| **`DDP_PHILOSOPHY.md` (este)** | Norte absoluto del producto. Qué es DDP y qué merece existir. |
| `COACH_PHILOSOPHY.md` | Cómo piensa y habla el Coach dentro de esta misión. |
| `DDP_RULES.md` | Reglas de diseño, convenciones y disciplina de ingeniería. |
| `ARCHITECTURE.md` y motores | Cómo se construye el sistema sin traicionar la misión. |
| `docs/project/` (PMS) | Memoria oficial del proyecto. Estado, decisiones, relevo, diario. |

La arquitectura y los motores son medios.
Los RFCs proponen caminos.
El PMS recuerda la historia.
**Esta filosofía es el criterio.**

---

## Lo que DDP nunca debe ser

DDP nunca debe convertirse en:

- un visor de gráficos
- un generador de estadísticas
- un panel lleno de métricas
- un sistema que impresione por la cantidad de datos
- una colección de pantallas técnicas sin utilidad para el piloto

Cada dato mostrado debe ayudar al piloto a **tomar una decisión**
o a **comprender mejor su evolución**.

Si un dato no cambia ninguna decisión del piloto o del Coach,
probablemente **no deba mostrarse**.

Este apartado es un filtro permanente:

antes de añadir una métrica, un gráfico, una tarjeta o una pantalla,
preguntar si el piloto sale más capaz de decidir o de entender su arco.
Si no, no entra en DDP.

---

## Lema del proyecto

**No analizamos vueltas.**

**Desarrollamos pilotos.**

**Cada decisión del sistema debe acercar al piloto a convertirse en un piloto más completo, más rápido y más inteligente.**

Ese es el ADN de DDP.
)
