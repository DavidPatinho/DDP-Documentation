# DDP — ChatGPT Context Pack
> Generated: 2026-08-08 · Product 4.0 · AICS-v2.1
>
> This file is the **offline/uploadable memory pack** for ChatGPT.
> Prefer uploading this file (or the ZIP) instead of asking ChatGPT to browse GitHub.

## Pack metadata

- Official Documentation Repository: `https://github.com/DavidPatinho/DDP-Documentation`
- Branch: `main`
- ZIP download URL: `https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.zip`
- Bundle download URL: `https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.md`

---

## Instructions for the AI

1. Read this entire pack.
2. Treat the PMS sections below as the only source of truth.
3. Follow **AI INITIALIZATION PROTOCOL** from the AI Bootstrap section.
4. Reconstruct project context, then wait for the user's next instruction.
5. Do not invent facts not present in this pack.

---

# ========== AI Bootstrap (AICS entry) ==========
# Source: docs/project/AI_BOOTSTRAP.md

# DDP — AI Bootstrap (AICS v2)

> **Entrada oficial para Inteligencias Artificiales.**
> Leer completo antes de cualquier tarea. Profundizar en el PMS solo si hace falta.
> Tiempo objetivo: **&lt; 1 minuto** para el resumen; protocolo completo antes de desarrollar.
>
> Humanos → [`BOOTSTRAP.md`](BOOTSTRAP.md).  
> Configuración global (incl. repo de docs) → [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md).  
> Fuente de verdad → **PMS**, nunca los chats.

**Actualizado:** 2026-08-08 · **Producto:** DDP 4.0 RC · **AICS:** v2.1 · **Siguiente fase:** Coach en tiempo real

---

## Qué es DDP

| | |
|--|--|
| **Qué** | Sistema de desarrollo de pilotos (simracing · escritorio) |
| **Misión** | Que el piloto se convierta en el mejor que puede llegar a ser (arco a largo plazo) |
| **Lema** | *No analizamos vueltas. Desarrollamos pilotos.* |
| **Filosofía** | Telemetría = medio. Producto = evolución del piloto. Confianza = evidencia + explicación. Entrenar con propósito &gt; acumular datos. |

Detalle: [`../DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md) · [`AI_GUIDE.md`](AI_GUIDE.md)

---

## Arquitectura

Capas de producto: `Tauri → Frontend (proyección) → Backend → Core → SQLite`.

Pipeline de responsabilidades (una por módulo):

```text
Import
  ↓
Analysis
  ↓
Interpretation
  ↓
Attribution
  ↓
Strategy Facts
  ↓
Development Strategy
  ↓
Planner
  ↓
Coach
  ↓
Dashboard
  ↓
Journey
```

| Módulo | Responsabilidad (solo qué, nunca cómo) |
|--------|----------------------------------------|
| **Import** | Entrada de hechos desde telemetría |
| **Analysis** | Medir: ¿qué ocurrió? |
| **Interpretation** | Diagnosticar la sesión: ¿qué priorizar hoy? |
| **Attribution** | Explicar causas: ¿por qué ocurrió? |
| **Strategy Facts** | Hechos estables para el programa |
| **Development Strategy** | Decidir el foco: ¿qué debo hacer ahora? |
| **Planner** | Materializar el entrenamiento de hoy |
| **Coach** | Comunicar / enseñar sin decidir |
| **Dashboard** | Proyectar el centro de mando |
| **Journey** | Narrar la memoria deportiva |

Infraestructura de desarrollo (no runtime del piloto):

```text
Development Infrastructure
  ↓
Project Memory System (PMS)
  ↓
AI Context System (AICS)
  ↓
Product Architecture
```

Detalle: [`../ARCHITECTURE.md`](../ARCHITECTURE.md)

---

## Estado actual

| Campo | Valor |
|-------|-------|
| **Versión** | DDP 4.0 |
| **Estado** | Release Candidate (producto offline usable) |
| **Arquitectura** | Pipeline 4.0 operativo · PMS + PMI + AICS v2 |
| **Motores congelados** | Import · Analysis · Interpretation · DATA_MODEL · DATABASE_DESIGN (docs v1.0) |
| **Frontend** | Dashboard A · Coach B · Explainability C · Journey D · Telemetría · Sesiones · Settings |
| **Backend** | Import / Analysis / Interpretation / Attribution / Strategy / Training · GET no persiste perfil |
| **Dashboard** | Foco = Strategy · causalidad = Attribution · proyecta, no interpreta |
| **Coach** | Narrative Intelligence · offline · no tiempo real |
| **Auditoría** | RCA-1 cerrada · 150 pytest |
| **Tiempo real** | No iniciado |
| **IA conversacional** | Futuro (voz, nunca cerebro) |
| **Próxima gran fase** | Coach en tiempo real / Ingeniero de pista |

Vivo: [`PROJECT_STATE.md`](PROJECT_STATE.md) · [`HANDOVER.md`](HANDOVER.md)

---

## Decisiones inmutables

Sin autorización expresa **no romper**:

- La **IA nunca decide**, nunca interpreta telemetría, nunca genera hechos.
- La **IA nunca modifica** Attribution ni Strategy.
- La **UI nunca interpreta** causalidad ni elige foco.
- **Strategy** decide el foco; **Planner** no cambia Strategy.
- **Coach comunica**; no es el cerebro.
- **Analysis** genera hechos; **Attribution** interpreta causas.
- Cronología por `session.started_at` (nunca orden de importación).
- **Historia &gt; sesión**; el foco no cambia por una sola tanda.
- **Una responsabilidad = un módulo**.
- **Dashboard proyecta**.
- **GET no escribe perfil**.
- Docs/esquema congelados: no tocar sin descongelación.
- DECISIONS / ADR: **append-only**.

Registro: [`ADR.md`](ADR.md) · [`DECISIONS.md`](DECISIONS.md)

---

## Problemas abiertos

| Prioridad | Resumen |
|-----------|---------|
| MEDIO | Huérfanos `.ibt` en disco tras delete (KI-001) |
| MEDIO | Política de `analysis_report` tras borrar sesión (KI-002) |
| BAJO | Deuda menor (legacy UI, stubs reports, drift sessionContext, …) |

Completo: [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md)

---

## Visión futura

```text
Ingeniero de pista (tiempo real)
  → Coach conversacional
  → AI Communication Layer
  → Mentor deportivo
```

IA = **voz** de DDP. Nunca su **cerebro**.  
[`ROADMAP.md`](ROADMAP.md) · [`AI_GUIDE.md`](AI_GUIDE.md)

---

## Cómo debe trabajar una IA

Antes de proponer cualquier cambio:

1. ¿Respeta `DDP_PHILOSOPHY`?
2. ¿Respeta `ARCHITECTURE`?
3. ¿Respeta `ADR`?
4. ¿Respeta `DECISIONS`?
5. ¿Rompe una decisión / documento congelado?

Si viola filosofía/arquitectura/ADR/DECISIONS o rompe algo congelado →
**detenerse y explicarlo**. No inventar hechos. No cerrar fases sin `/close-phase`.

Prioridad: `PHILOSOPHY → ARCHITECTURE → ADR → DECISIONS → STATE → HANDOVER`.

---

## AI INITIALIZATION PROTOCOL

Procedimiento **obligatorio** para cualquier IA (Cursor, ChatGPT u otra):

1. Leer **este** documento (`AI Bootstrap Path` en
   [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md)) completamente.
2. **ChatGPT:** no navegar GitHub. Usar el **Context Pack**
   (`Context Pack Download URL` / archivo adjunto por el usuario).
   **Cursor / entorno con repo local:** leer el PMS en disco.
   Campos solo desde [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md).
3. Leer obligatoriamente (en el pack o en el PMS):
   - `DDP_PHILOSOPHY.md`
   - `ARCHITECTURE.md`
   - `PROJECT_CONTEXT.md`
   - `PROJECT_STATE.md`
   - `DECISIONS.md`
   - `ADR.md`
   - `HANDOVER.md`
   - `KNOWN_ISSUES.md`
4. Reconstruir mentalmente el proyecto (pipeline, estado, invariantes, siguiente fase).
5. Considerar el **PMS** la única fuente oficial de verdad.
6. **No** comenzar ninguna tarea de desarrollo hasta comprender el contexto.
7. Si existe contradicción entre una conversación y el PMS → **prevalece el PMS**.

Cierre de fase: `py -m tools.pms.close_phase --phase "…"`.  
Spec: [`PMS_SPEC.md`](PMS_SPEC.md).

---

## ChatGPT Startup Prompt

> **PROMPT OFICIAL** — copiar y pegar en ChatGPT.  
> Generado **automáticamente** desde [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md).  
> **No editar a mano.** Regenerar:
>
> ```bash
> py -m tools.pms.update_ai_bootstrap --refresh-prompt --apply
> ```

<!-- AICS:CHATGPT_PROMPT_BEGIN -->
```text
Eres una IA trabajando en el proyecto DDP (Driver Development Program).
Versión de producto: 4.0 · PMS/AICS: AICS-v2.1

IMPORTANTE: No intentes navegar GitHub por tu cuenta. El usuario te proporciona
el paquete de contexto (archivo adjunto o enlace de descarga).

1. Usa el Context Pack oficial (configuración del proyecto):
   - Markdown (recomendado subir este archivo): https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.md
   - ZIP: https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.zip
   - Rutas en el repo de docs: docs/chatgpt/DDP_AICS_CONTEXT.md · docs/chatgpt/DDP_AICS_CONTEXT.zip
   Si el usuario ya ha adjuntado el archivo, úsalo directamente sin descargar.

2. Lee completamente el Context Pack (incluye AI_BOOTSTRAP y los documentos
   obligatorios del PMS).

3. Sigue al pie de la letra la sección "AI INITIALIZATION PROTOCOL"
   (DDP_PHILOSOPHY, ARCHITECTURE, PROJECT_CONTEXT, PROJECT_STATE, DECISIONS,
   ADR, HANDOVER, KNOWN_ISSUES).

4. Consulta únicamente el contenido del pack / PMS cuando necesites profundizar.
   No uses conversaciones anteriores como fuente de verdad.
   Repo de documentación (referencia, no navegación obligatoria):
   https://github.com/DavidPatinho/DDP-Documentation (rama main)

5. Reconstruye completamente el contexto del proyecto (misión, pipeline, estado,
   decisiones inmutables, problemas abiertos, próxima gran fase).

6. Cuando el contexto esté reconstruido, confirma brevemente que estás lista
   y espera la siguiente instrucción del usuario.
   No propongas cambios ni escribas código hasta que el usuario lo pida.
```
<!-- AICS:CHATGPT_PROMPT_END -->

---

*AICS v2.1 · prompt desacoplado de la URL · conocimiento en el proyecto.*


---

# ========== Philosophy ==========
# Source: docs/DDP_PHILOSOPHY.md

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


---

# ========== Architecture ==========
# Source: docs/ARCHITECTURE.md

# DDP — Arquitectura

> Documento de arquitectura del sistema.
>
> **Norte de producto:** `DDP_PHILOSOPHY.md` — qué es DDP y qué merece existir.
> Complementa `DDP_RULES.md` (diseño, convenciones y disciplina de ingeniería)
> y `COACH_PHILOSOPHY.md` (cómo piensa el Coach).
>
> **Memoria del proyecto:** `docs/project/` — PMS + AICS.
> Forman parte oficial de la arquitectura de desarrollo. No son opcionales.
>
> Ante conflicto de *misión o criterio de producto*, manda `DDP_PHILOSOPHY.md`.
> Ante conflicto de *reglas técnicas o de UI*, manda `DDP_RULES.md`.
> Ante conflicto de *estado actual / relevo*, mandan `docs/project/PROJECT_STATE.md`
> y `docs/project/HANDOVER.md`.

---

## Capas del sistema (visión completa)

```text
Development Infrastructure          tools/ · scripts de desarrollo
        ↓
Project Memory System (PMS)         docs/project/ — memoria oficial
        ↓
AI Context System (AICS)            BOOTSTRAP / AI_BOOTSTRAP / config
        ↓
Product Architecture                Tauri → Frontend → Backend → Core → SQLite
```

El AICS pertenece a la **infraestructura oficial de desarrollo**.
No forma parte del runtime del piloto.

---

## Visión general (Product Architecture)

DDP se organiza en capas con responsabilidades estrictamente separadas.
El elemento central es el **Core**: un motor de dominio en Python que no
depende del frontend ni del backend.

```
┌──────────────────────────────────────────────────────────┐
│  TAURI — shell nativo de escritorio (Windows)            │
│  Ventana, ciclo de vida, acceso a sistema de archivos    │
└──────────────────────────────────────────────────────────┘
                            │
┌──────────────────────────────────────────────────────────┐
│  FRONTEND — React + TypeScript + Tailwind + shadcn/ui    │
│  Presentación, navegación, interacción, visualización    │
│  NO contiene lógica de dominio · NO interpreta causalidad│
└──────────────────────────────────────────────────────────┘
                            │  HTTP / JSON
┌──────────────────────────────────────────────────────────┐
│  BACKEND — Python FastAPI                                │
│  Endpoints, validación, orquestación, persistencia       │
│  NO contiene lógica de dominio: la delega al Core        │
└──────────────────────────────────────────────────────────┘
                            │  llamadas Python directas
┌──────────────────────────────────────────────────────────┐
│  CORE — motores deterministas (Pandas / NumPy)           │
│  Telemetría, análisis, interpretación, attribution,      │
│  strategy, training · Independiente de HTTP, UI y BD     │
└──────────────────────────────────────────────────────────┘
                            │
┌──────────────────────────────────────────────────────────┐
│  SQLITE — persistencia local                             │
└──────────────────────────────────────────────────────────┘
```

Pipeline oficial de producto (DDP 4.0):

```text
Import → Analysis → Interpretation
       → Attribution → Strategy Facts → Development Strategy
       → Planner → Coach → Dashboard → Journey
```

Una pregunta por motor (ver `COACH_PHILOSOPHY.md`):

| Motor | Pregunta |
|-------|----------|
| Analysis | ¿Qué ocurrió? |
| Interpretation | ¿Qué problema priorizar en esta sesión? |
| Attribution | ¿Por qué ocurrió? |
| Development Strategy | ¿Qué debo hacer ahora? |
| Training Planner | ¿Cómo entreno hoy? |
| Progress / Profile | ¿Ha funcionado? ¿Cómo evoluciono? |
| Coach / Dashboard / Journey | ¿Cómo se lo digo al piloto sin contradecir los hechos? |

---

## Project Memory System (PMS) + AICS + PMI

El PMS es la **memoria oficial** de DDP. Vive en `docs/project/`.

La **PMI** (`tools/pms/`) valida y orquesta el mantenimiento del PMS.

El **AICS** (AI Context System) es la puerta de entrada:

| Audiencia | Entrada |
|-----------|---------|
| Humanos | `docs/project/BOOTSTRAP.md` |
| IA | `docs/project/AI_BOOTSTRAP.md` |

El AICS no sustituye al PMS: resume y enlaza para reconstrucción rápida.
Especificación: `docs/project/PMS_SPEC.md`. Herramientas: `tools/README.md`.

| Documento | Función |
|-----------|---------|
| `BOOTSTRAP.md` | Entrada humana (AICS) |
| `AI_BOOTSTRAP.md` | Entrada IA + protocolo + ChatGPT Startup Prompt |
| `PROJECT_CONFIGURATION.md` | Configuración global (única fuente de URLs/repos) |
| `PROJECT_CONTEXT.md` | Incorporación |
| `PROJECT_STATE.md` | Estado actual |
| `DECISIONS.md` | Decisiones de producto |
| `ADR.md` | Decisiones técnicas |
| `ROADMAP.md` | Evolución de producto |
| `KNOWN_ISSUES.md` | Incidencias abiertas |
| `HANDOVER.md` | Relievo (única versión viva) |
| `SESSION_NOTES.md` | Diario cronológico (append-only) |
| `CHANGELOG_AI.md` | Cambios funcionales |
| `AI_GUIDE.md` | Papel de la IA |
| `PMS_SPEC.md` | Especificación oficial PMS/PMI/AICS |
| `PHASE_CLOSE_CHECKLIST.md` | Checklist reutilizable de cierre |

### Procedimiento oficial: `/close-phase`

```text
Desarrollar → Tests → validate_pms → actualizar PMS
→ HANDOVER → Issues → STATE → CHANGELOG → ADR/DECISIONS
→ AICS (AI_BOOTSTRAP + ChatGPT prompt desde PROJECT_CONFIGURATION)
→ sync_docs → validación final → CERRAR
```

```bash
py -m tools.pms.close_phase --phase "Nombre de la fase"
py -m tools.pms.validate_pms
```

Ninguna fase se considera terminada hasta que el procedimiento complete.
Código verde + tests verdes **no bastan**.

### Orden de arranque

- **IA:** `AI_BOOTSTRAP.md` → profundizar solo con enlaces del PMS.
- **Humano:** `BOOTSTRAP.md` → luego lectura profunda:

1. `DDP_PHILOSOPHY.md`
2. `ARCHITECTURE.md` (este)
3. `docs/project/PROJECT_CONTEXT.md`
4. `docs/project/PROJECT_STATE.md`
5. `docs/project/DECISIONS.md`
6. `docs/project/ADR.md`
7. `docs/project/HANDOVER.md`
8. `docs/project/KNOWN_ISSUES.md`

### Papel de la IA

La IA es la **voz** de DDP, nunca su **cerebro**.
Detalle normativo: `docs/project/AI_GUIDE.md` y ADR-002.

```text
Motores deterministas → AI Communication Layer → Conversation
                     → Real Time Engineer → Mentor Deportivo
```

### Futuro: documentación pública

```text
Repo privado (código + docs/) → sync_docs / publish_docs → repo público (solo docs)
```

Preparado en PMI; publicación remota **no** implementada en v1.

---

## El Core como motor independiente

El Core es el cerebro de DDP y la pieza de mayor valor a largo plazo. Contiene
todo el conocimiento de dominio del producto: cómo se lee la telemetría, cómo
se mide el rendimiento, cómo se detecta una limitación y cómo se convierte en
una recomendación útil.

### Independencia estricta

El Core **no conoce ni importa** ninguna de estas tecnologías:

| No depende de | Motivo |
|---------------|--------|
| FastAPI, HTTP, requests | El transporte es una decisión externa al dominio |
| React, DOM, Tauri | La presentación es intercambiable |
| SQLite, ORMs, sesiones de BD | La persistencia es responsabilidad del backend |

Solo depende de **Pandas**, **NumPy** y la biblioteca estándar de Python.

### Dirección de dependencias

Las dependencias apuntan siempre hacia dentro. El Core es el centro y nunca
mira hacia fuera.

```
frontend  ──>  backend  ──>  core
```

El backend importa el Core. El Core nunca importa el backend.

### Qué habilita esta independencia

- **Testeable en aislamiento** — se ejecuta sin levantar servidor ni interfaz
- **Reutilizable** — accesible desde scripts, notebooks o herramientas CLI
- **Sustituible en los extremos** — cambiar de framework web o de interfaz no
  obliga a reescribir lógica de dominio
- **Longevidad** — la lógica de análisis sobrevive a los ciclos de vida de los
  frameworks que la rodean
- **Razonamiento aislado** — un cambio en el análisis de neumáticos no puede
  romper una ruta HTTP

### Contrato del Core

| Aspecto | Definición |
|---------|------------|
| Entrada | Datos y parámetros como estructuras Python o DataFrames |
| Salida | Estructuras de datos puras (dicts, dataclasses, DataFrames) |
| Efectos | Ninguno: no escribe en BD, no hace red, no toca la UI |
| Errores | Excepciones de dominio propias, nunca códigos HTTP |

El Core nunca devuelve HTML, JSON serializado ni componentes. Devolver datos
puros permite que el mismo resultado se renderice en pantalla, se exporte a
PDF o se consuma desde un script sin duplicar lógica.

---

## Estructura del Core

```
core/
├── telemetry/              Lectura y análisis de datos de pista
│   ├── parser.py / iracing/  Ingesta .ibt y normalización
│   ├── analyzer.py         Medición de sesión
│   └── …                   Consistencia, series, corners, etc.
│
├── engineer/               Interpretation Engine (sesión)
│   ├── coach.py            Diagnóstico e insights
│   └── recommendations.py  Acciones con success_criterion
│
├── attribution/            Performance Attribution (multi-sesión)
│   ├── engine.py           ¿Por qué cambió el rendimiento?
│   └── models.py           PerformanceAttribution
│
├── development_strategy/   Development Strategy Engine
│   └── engine.py           ¿Qué debo hacer ahora? (on-demand)
│
├── training/               Skills, planner, progress, perfil, circuit map
│
├── reports/                Composición de informes (session; weekly/career stubs)
├── scoring/                Driver score (evolución futura)
├── objectives/             Objetivos
└── utils/                  Utilidades transversales
```

### Flujo de procesamiento

```
   fichero de telemetría
            │
            ▼
   telemetry/                 normaliza y mide (Analysis)
            │
            ▼
   engineer/                  diagnostica sesión (Interpretation)
            │
            ▼
   training/*                 skills / plan / progress / profile
            │
            ▼
   attribution/               explica causas (PerformanceAttribution)
            │
            ▼
   development_strategy/      decide foco (DevelopmentStrategy)
            │
            ▼
   backend  ──>  frontend     serializa y proyecta (Coach / Dashboard / Journey)
```

Dashboard y Coach **no** inventan causalidad ni foco: proyectan
`PerformanceAttribution` y `DevelopmentStrategy`.

### Reglas internas del Core

1. Un módulo tiene una responsabilidad y la documenta en su docstring
2. `telemetry/` mide, `engineer/` diagnostica sesión, `attribution/` explica,
   `development_strategy/` decide foco, `training/` planifica entrenamiento —
   sin mezclar
3. `utils/` solo admite lo compartido por dos o más módulos
4. Ningún módulo del Core accede a base de datos ni a red
5. Los límites de tamaño de `DDP_RULES.md` aplican igual que en el frontend
6. Planner **nunca** cambia Strategy (ADR-008)
7. Cronología por `session.started_at`, nunca por orden de importación (ADR-005)

---

## Responsabilidades por capa

### Tauri

Contenedor nativo. Gestiona ventana, ciclo de vida de la aplicación y acceso
al sistema de archivos. No contiene lógica de negocio.

### Frontend (`frontend/`)

Presentación e interacción exclusivamente.

- Renderiza datos que recibe del backend
- Gestiona navegación y estado de vista
- Proyecta experiencias (Dashboard, Coach, Explainability, Journey)

**No hace:** procesar telemetría, calcular métricas, decidir foco, reinterpretar
dimensiones de Attribution cuando el motor falla.

### Backend (`backend/`)

Frontera entre el mundo exterior y el Core.

- Expone endpoints HTTP y valida entradas
- Lee y escribe en SQLite
- Llama al Core y serializa sus resultados
- Gestiona ficheros subidos y errores de transporte

**No hace:** analizar telemetría ni generar recomendaciones de dominio.
Lecturas GET de training **no** persisten perfil (ADR-013).

### Core (`core/`)

Toda la lógica de dominio. Detallado en las secciones anteriores.

### Database (`database/`)

Ficheros SQLite y migraciones 0001–0010. Accedido únicamente desde el backend.
Diseño congelado: `DATABASE_DESIGN.md` v1.0.

### Project memory (`docs/project/`)

Memoria oficial del proyecto (PMS). Obligatorio actualizar al cerrar fases.

---

## Frontera entre backend y Core

El backend actúa como adaptador. Un endpoint típico sigue este patrón:

```
1. Recibe la petición HTTP y valida el payload
2. Lee de SQLite los datos necesarios
3. Llama al Core con estructuras Python puras
4. Recibe estructuras Python puras del Core
5. Persiste lo que deba persistirse
6. Serializa la respuesta a JSON
```

Los pasos 1, 2, 5 y 6 son del backend. El paso 3–4 es del Core. Esta frontera
no debe difuminarse: en cuanto el Core recibe un objeto de FastAPI o una sesión
de base de datos, deja de ser independiente.

---

## Estructura completa del proyecto

```
DDP/
├── core/          Motores de dominio independientes (Python)
├── backend/       API FastAPI — adaptador HTTP sobre el Core
├── frontend/      React + Tauri — proyección
├── database/      SQLite y migraciones
├── docs/          Documentación del proyecto
│   └── project/   Project Memory System (PMS) + PMS_SPEC
├── tools/         Infraestructura de desarrollo
│   └── pms/       Project Memory Infrastructure (PMI v1)
└── assets/        Recursos compartidos (p. ej. training YAML)
```

---

## Estado actual

| Capa | Estado |
|------|--------|
| Frontend | Operativo — Dashboard A, Coach B, Explainability C, Journey D, Telemetría, Sesiones, Settings |
| Backend | Operativo — import, analysis, interpretation, attribution, strategy, training, active driver |
| Core | Operativo — telemetry, engineer, attribution, development_strategy, training; reports weekly/career stubs |
| Database | Migraciones 0001–0010 · diseño congelado |
| Tauri | Configurado · shell de escritorio |
| PMS | Memoria oficial en `docs/project/` |
| PMI | **v1** · `tools/pms/` · `/close-phase` |
| AICS | **v2.1** · GitHub docs + prompt auto desde `PROJECT_CONFIGURATION` |

Producto: **DDP 4.0 Release Candidate** (post RCA-1 + PMI + AICS v2.1).  
Detalle vivo: `docs/project/PROJECT_STATE.md`.  
Relievo: `docs/project/HANDOVER.md`.  
Entrada IA: `docs/project/AI_BOOTSTRAP.md`.  
Config: `docs/project/PROJECT_CONFIGURATION.md`.  
Spec: `docs/project/PMS_SPEC.md`.

Documentos de motor congelados (no modificar sin descongelación explícita):

- `IMPORT_PIPELINE.md` v1.0
- `ANALYSIS_ENGINE.md` v1.0
- `INTERPRETATION_ENGINE.md` v1.0
- `DATA_MODEL.md` / `DATABASE_DESIGN.md` v1.0

---

## Flujo de mantenimiento del PMS (oficial)

El mantenimiento ya no es solo manual. Procedimiento:

```bash
py -m tools.pms.close_phase --phase "…"
```

1. Desarrollar respetando filosofía, arquitectura y ADRs.
2. Tests.
3. `validate_pms`.
4. Actualizar PMS (STATE, HANDOVER, SESSION_NOTES, …).
5. ADR / DECISIONS si proceden (append-only).
6. AICS: actualizar `AI_BOOTSTRAP.md` si cambió el contexto general.
7. `sync_docs`.
8. Validación final → cerrar o permanecer ABIERTA.

Detalle: `docs/project/PMS_SPEC.md` · checklist:
`docs/project/PHASE_CLOSE_CHECKLIST.md`.

---

*Documento vivo. Actualizar cuando cambie la estructura de capas, el pipeline
de motores o el contrato entre ellas. El estado operativo detallado vive en
el PMS (`PROJECT_STATE.md`).*


---

# ========== Project Configuration ==========
# Source: docs/project/PROJECT_CONFIGURATION.md

# DDP — Project Configuration

> **Única fuente de configuración global** del PMS / AICS / PMI.
>
> Ningún otro documento debe duplicar estos valores (URLs, rama, rutas).
> Si cambia el repositorio de documentación, **solo se edita este archivo**;
> después regenera prompt + context pack:
>
> ```bash
> py -m tools.pms.generate_context_pack --apply
> py -m tools.pms.update_ai_bootstrap --refresh-prompt --apply
> ```

**Última actualización:** 2026-08-08

---

## Identity

Project Name:
DDP

Current Version:
4.0

PMS Version:
AICS-v2.1

---

## Official Documentation Repository

Official Documentation Repository:
https://github.com/DavidPatinho/DDP-Documentation

Repository Branch:
main

Repository Docs Path:
docs/

---

## AICS Paths

AI Bootstrap Path:
docs/project/AI_BOOTSTRAP.md

Bootstrap Path:
docs/project/BOOTSTRAP.md

---

## ChatGPT Context Pack

> ChatGPT normalmente **no puede navegar GitHub**. Usa este paquete.

Context Pack Path:
docs/chatgpt/DDP_AICS_CONTEXT.md

Context Pack Zip Path:
docs/chatgpt/DDP_AICS_CONTEXT.zip

Context Pack Download URL:
https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.zip

Context Pack Markdown Download URL:
https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.md

---

## Rules

1. **Single source:** URL, rama, rutas y enlaces de pack solo aquí.
2. **No duplication:** ningún otro documento escribe esas URLs a mano (excepto el prompt generado).
3. Regenerar pack + prompt tras cambios de contexto:
   `generate_context_pack` → `update_ai_bootstrap` → `publish_docs --prepare` → push docs repo.
4. Para ChatGPT: el usuario **descarga/sube** el context pack; no hace falta que ChatGPT entre en GitHub.

---

*Configuración global · no es narrativa de producto.*


---

# ========== Project Context ==========
# Source: docs/project/PROJECT_CONTEXT.md

# DDP — Project Context

> Documento de incorporación al proyecto.
> Debe permitir comprender DDP en menos de cinco minutos.
>
> Parte del **Project Memory System (PMS v1)**.
> Lectura obligatoria al inicio de cualquier conversación nueva
> (después de `DDP_PHILOSOPHY.md` y `ARCHITECTURE.md`).

**Última actualización:** 2026-08-08

---

## Qué es DDP

**Driver Development Program** — sistema de desarrollo de pilotos de simracing
(aplicación de escritorio, Tauri + React + FastAPI + Core Python + SQLite).

No es un visor de telemetrías ni un generador de informes.
La telemetría es la materia prima; el producto es el **piloto que mejora**.

Lema:

> **No analizamos vueltas. Desarrollamos pilotos.**

Detalle: [`docs/DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md).

---

## Filosofía

1. La telemetría es un medio, nunca el objetivo.
2. Cada sesión es un capítulo de una historia mayor.
3. Entrenar con propósito > acumular datos.
4. Toda recomendación debe poder justificarse con evidencia.
5. El Coach enseña; no solo analiza.
6. Pensar en el piloto de dentro de seis meses.

**Confianza** = evidencia + explicación. Sin eso, no hay desarrollo real.

---

## Arquitectura general

Cuatro capas con dependencias hacia dentro:

```text
Tauri (shell)
  → Frontend (React · proyección, nunca decide)
    → Backend (FastAPI · orquestación / persistencia)
      → Core (motores deterministas · dominio)
        → SQLite
```

Pipeline oficial de producto (DDP 4.0):

```text
Import → Analysis → Interpretation
       → Attribution → Strategy Facts → Development Strategy
       → Planner → Coach → Dashboard → Journey
```

Detalle: [`docs/ARCHITECTURE.md`](../ARCHITECTURE.md).

---

## Motores existentes

| Motor | Pregunta | Estado |
|-------|----------|--------|
| **Import Pipeline** | ¿Qué hechos entran? | Implementado · doc v1.0 congelada |
| **Analysis Engine** | ¿Qué ocurrió? | Implementado · doc v1.0 congelada |
| **Interpretation Engine** | ¿Qué problema priorizar en esta sesión? | Implementado · doc v1.0 congelada |
| **Performance Attribution** | ¿Por qué ocurrió? | Implementado · consumido por Dashboard/Coach |
| **Strategy Facts** | Hechos estables para Strategy | Implementado · hidrata desde SQLite |
| **Development Strategy** | ¿Qué debo hacer ahora? | Implementado · on-demand, sin persistir |
| **Training Planner** | ¿Cómo entreno hoy? | Implementado · assignments / ejercicios |
| **Progress / Profile** | ¿Ha funcionado? ¿Cómo evoluciono? | Implementado · dual timeline |
| **Coach / Dashboard / Journey** | ¿Cómo se lo digo al piloto? | Proyección UI · Fases A–D + Narrative Intelligence |

---

## Qué está congelado

No modificar sin proceso explícito de descongelación:

| Ámbito | Documento / alcance |
|--------|---------------------|
| Arquitectura de capas | `ARCHITECTURE.md` (contrato Core) |
| Modelo de datos | `DATA_MODEL.md` v1.0 |
| Diseño BD | `DATABASE_DESIGN.md` v1.0 · migraciones 0001–0010 |
| Import | `IMPORT_PIPELINE.md` v1.0 |
| Analysis | `ANALYSIS_ENGINE.md` v1.0 |
| Interpretation | `INTERPRETATION_ENGINE.md` v1.0 |
| Driver Experience A–C | Dashboard · Coach briefing · Explainability (proyección) |
| Fase D Journey | Memoria deportiva (`/driver`) — entregada; no reabrir sin motivo |

Los motores de dominio **no** se reescriben desde la UI.
La UI proyecta; no calcula foco ni causalidad.

---

## Documentos obligatorios (PMS)

| Documento | Rol |
|-----------|-----|
| [`BOOTSTRAP.md`](BOOTSTRAP.md) | Entrada humana (AICS) |
| [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) | Entrada IA + protocolo + prompt ChatGPT |
| [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md) | Config global (repos/URLs) |
| [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) | Incorporación (este) |
| [`PROJECT_STATE.md`](PROJECT_STATE.md) | Estado actual visual |
| [`DECISIONS.md`](DECISIONS.md) | Decisiones de producto |
| [`ADR.md`](ADR.md) | Decisiones técnicas |
| [`ROADMAP.md`](ROADMAP.md) | Evolución de producto |
| [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md) | Incidencias abiertas |
| [`HANDOVER.md`](HANDOVER.md) | Relievo de fase |
| [`SESSION_NOTES.md`](SESSION_NOTES.md) | Diario cronológico |
| [`CHANGELOG_AI.md`](CHANGELOG_AI.md) | Cambios funcionales |
| [`AI_GUIDE.md`](AI_GUIDE.md) | Papel de la IA en DDP |
| [`PMS_SPEC.md`](PMS_SPEC.md) | Especificación oficial PMS + PMI + AICS |
| [`PHASE_CLOSE_CHECKLIST.md`](PHASE_CLOSE_CHECKLIST.md) | Checklist de cierre |

**Entrada:** humanos → `BOOTSTRAP.md` · IA → `AI_BOOTSTRAP.md`  
**Infraestructura:** `tools/pms/` — procedimiento `/close-phase`  
**Norte de producto:** `docs/DDP_PHILOSOPHY.md`  
**Reglas de ingeniería:** `docs/DDP_RULES.md`  
**Coach:** `docs/COACH_PHILOSOPHY.md`

---

## Estado general

| Campo | Valor |
|-------|-------|
| Versión de producto | **DDP 4.0** |
| Estado | **Release Candidate** (post RCA-1) |
| Tests | 150 pytest en verde (auditoría RCA-1) |
| Última fase cerrada | AICS v2 (protocolo + ChatGPT Startup Prompt) |
| Próxima gran fase | **Coach en tiempo real / Ingeniero de pista** |

El producto offline es usable: importar → analizar → incorporar al perfil →
Dashboard / Coach / Explainability / Trayectoria con una sola historia por
piloto activo.

---

## Próxima gran fase

```text
Coach en tiempo real
        ↓
Ingeniero de pista
        ↓
IA conversacional (voz de DDP, nunca cerebro)
        ↓
Mentor deportivo
```

La IA **comunica, explica, enseña, resume, conversa y acompaña**.
Nunca decide, interpreta telemetría, genera hechos ni modifica Attribution
o Strategy. Ver [`AI_GUIDE.md`](AI_GUIDE.md).

---

## Orden de lectura al incorporar

**Primero:** [`BOOTSTRAP.md`](BOOTSTRAP.md) (humano) o
[`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) (IA).

Luego (profundidad):

1. `docs/DDP_PHILOSOPHY.md`
2. `docs/ARCHITECTURE.md`
3. `docs/project/PROJECT_CONTEXT.md` ← estás aquí
4. `docs/project/PROJECT_STATE.md`
5. `docs/project/DECISIONS.md`
6. `docs/project/ADR.md`
7. `docs/project/HANDOVER.md`
8. `docs/project/KNOWN_ISSUES.md`

Solo después: desarrollo.

---

*PMS v1 · memoria oficial del proyecto.*


---

# ========== Project State ==========
# Source: docs/project/PROJECT_STATE.md

# DDP — Project State

> Estado actual del proyecto. Debe mantenerse siempre al día.
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08

---

## Resumen visual

```text
╔══════════════════════════════════════════════════════════════╗
║  DDP 4.0 · RELEASE CANDIDATE                                 ║
║  PMS + PMI v1 + AICS v2.1                                    ║
║  Siguiente: Coach en tiempo real                             ║
╚══════════════════════════════════════════════════════════════╝
```

| Campo | Valor |
|-------|-------|
| **Versión** | DDP 4.0 |
| **Estado** | Release Candidate usable (producto offline) |
| **Fase actual** | Post AICS v2.1 · preparación Coach tiempo real |
| **Última fase terminada** | AICS v2.1 — GitHub Documentation Integration |
| **Tests** | 150 pytest ✅ (RCA-1) |
| **Última actualización** | 2026-08-08 |

---

## Arquitectura

```text
Import → Analysis → Interpretation
       → Attribution → Strategy Facts → Development Strategy
       → Planner → Coach → Dashboard → Journey
```

| Capa | Tecnología | Rol |
|------|------------|-----|
| Shell | Tauri | Ventana / archivos |
| Frontend | React + TS + Tailwind | Proyección · no decide |
| Backend | FastAPI | Orquestación · SQLite |
| Core | Python · Pandas/NumPy | Motores deterministas |
| DB | SQLite · migraciones 0001–0010 | Persistencia local |

---

## Motores

| Motor | Estado | Congelado |
|-------|--------|-----------|
| Import Pipeline | ✅ Operativo | Doc v1.0 |
| Analysis Engine | ✅ Operativo | Doc v1.0 |
| Interpretation Engine | ✅ Operativo | Doc v1.0 |
| Performance Attribution | ✅ Operativo | Arquitectura estable |
| Strategy Facts | ✅ Operativo | — |
| Development Strategy | ✅ Operativo (on-demand) | Diseño RFC 003 |
| Training Planner | ✅ Operativo | — |
| Progress / Profile | ✅ Operativo | — |
| Scoring / DriverIndex | ⏳ Stub / futuro | — |
| Reports weekly/career | ⏳ Stubs | — |

---

## Frontend

| Pantalla | Estado | Notas |
|----------|--------|-------|
| Dashboard | ✅ Fase A congelada | Centro de mando · «¿Qué necesito saber hoy?» |
| Coach (briefing) | ✅ Fase B + Narrative Intelligence | 4 actos · hechos → paso |
| Explainability | ✅ Fase C congelada | Sheet «¿Por qué?» |
| Trayectoria `/driver` | ✅ Fase D | Memoria deportiva / hitos |
| Telemetría | ✅ | Import · incorporate · delete |
| Sesiones | ✅ | Analysis + Interpretation |
| Settings | ✅ | Selector de piloto activo |
| Informes (nav) | ⏸ Fuera del nav RC | Stub retirado (RCA-1 M1) |

---

## Backend

| Área | Estado |
|------|--------|
| Import API | ✅ |
| Analysis API | ✅ |
| Interpretation API | ✅ |
| Attribution API | ✅ |
| Strategy / Facts | ✅ |
| Training / Profile | ✅ · GET no persiste (RCA-1) |
| Active driver | ✅ `GET/PUT /drivers/active` |
| Reset / Dev tools | ✅ |

---

## Dashboard

| Aspecto | Estado |
|---------|--------|
| Fuente de foco | **Strategy** (nunca `weaknesses[0]`) |
| Causalidad | **Attribution** (UI no reinterpretá) |
| Invalidación | Evento `ddp:product-mutated` |
| Piloto | Filtrado por piloto activo |

---

## Coach

| Aspecto | Estado |
|---------|--------|
| Voz | Narrative Intelligence · prosa de pista |
| Estructura | HECHO → CONSECUENCIA → INTERPRETACIÓN → PRÓXIMO PASO |
| Memoria | Anclas temporales cuando hay evidencia |
| Tiempo real | ❌ Pendiente (próxima gran fase) |

---

## Auditoría

| Aspecto | Estado |
|---------|--------|
| RCA-1 | ✅ Cerrada 2026-08-08 |
| Criterio | Base sólida para Coach tiempo real |
| Deudas abiertas | Ver [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md) |

---

## Tiempo real

| Capacidad | Estado |
|-----------|--------|
| Coach durante sesión | ❌ No iniciado |
| Ingeniero de pista live | ❌ Visión |
| Streaming telemetría live | ❌ Fuera de alcance actual |

---

## IA Conversacional

| Capacidad | Estado |
|-----------|--------|
| Capa de comunicación | ❌ Futuro |
| Conversation Engine | ❌ Futuro |
| Mentor deportivo | ❌ Visión |

Reglas: [`AI_GUIDE.md`](AI_GUIDE.md) — la IA es la **voz**, nunca el **cerebro**.

---

## PMS / PMI / AICS

| Componente | Estado |
|------------|--------|
| PMS (`docs/project/`) | ✅ Memoria oficial |
| PMI (`tools/pms/`) | ✅ v1 |
| AICS | ✅ **v2.1** — GitHub docs + prompt auto |
| `PROJECT_CONFIGURATION.md` | ✅ URL · rama · rutas · identidad |
| `PMS_SPEC.md` | ✅ Spec PMS/PMI/AICS v2 |
| `/close-phase` | ✅ Regenera prompt AICS |
| `validate_pms` | ✅ Check AICS + sync prompt |
| Repo público docs | ⏳ Marcador en PROJECT_CONFIGURATION |

---

## Enlaces

- Entrada humana: [`BOOTSTRAP.md`](BOOTSTRAP.md)
- Entrada IA: [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md)
- Config: [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md)
- Spec: [`PMS_SPEC.md`](PMS_SPEC.md)
- Relievo: [`HANDOVER.md`](HANDOVER.md)
- Roadmap: [`ROADMAP.md`](ROADMAP.md)
- Issues: [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md)
- Tools: [`../../tools/README.md`](../../tools/README.md)

---

*Actualizar este documento en cada cierre de fase.*


---

# ========== Decisions ==========
# Source: docs/project/DECISIONS.md

# DDP — Decisiones de producto

> Registro permanente de decisiones de producto.
> No confundir con [`ADR.md`](ADR.md) (decisiones técnicas de arquitectura).
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08

---

## D-001 — DDP desarrolla pilotos, no analiza vueltas

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Misión: desarrollo de pilotos |
| **Descripción** | El producto es la evolución del piloto a largo plazo. La telemetría es materia prima, no el destino. |
| **Motivación** | Evitar que DDP derive en visor de gráficos o panel de métricas. |
| **Impacto** | Toda funcionalidad se filtra por «¿ayuda al piloto a ser más completo, rápido e inteligente?» |
| **Estado** | Vigente |
| **Documentos** | `DDP_PHILOSOPHY.md` |
| **RFC** | — |

---

## D-002 — Confianza mediante evidencia y explicación

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Sin explicación no hay confianza |
| **Descripción** | Toda recomendación, mantenimiento o cambio de foco debe poder justificarse. El piloto puede preguntar «¿por qué?» |
| **Motivación** | La confianza es un pilar; sin ella el Coach se ignora. |
| **Impacto** | Explainability (Fase C); evidencia en Strategy/Attribution; rechazo de consejos opacos. |
| **Estado** | Vigente |
| **Documentos** | `DDP_PHILOSOPHY.md`, `DRIVER_EXPERIENCE_PHASE_C.md` |
| **RFC** | — |

---

## D-003 — El Coach no cambia de objetivo por una sola sesión

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Estabilidad de foco de entrenamiento |
| **Descripción** | El foco solo cambia con evidencia suficiente. Una sesión mala/buena no basta. Una skill nueva solo acumula evidencia. |
| **Motivación** | Comportarse como entrenador real, no como algoritmo reactivo. |
| **Impacto** | Development Strategy; jerarquía deportiva → transferencia → skills → métricas. |
| **Estado** | Vigente |
| **Documentos** | `COACH_PHILOSOPHY.md`, `DEVELOPMENT_STRATEGY_ENGINE.md` |
| **RFC** | RFC 003 |

---

## D-004 — Maximizar retorno del entrenamiento, no perfeccionar todas las skills

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Entrenar el limitante actual |
| **Descripción** | Se entrena lo que más limita el rendimiento ahora. Una skill abierta que ya no limita puede esperar. |
| **Motivación** | Evitar programas que intentan «cerrar el catálogo» en lugar de ganar tiempo en pista. |
| **Impacto** | `why_not_other` obligatorio; Strategy elige foco único. |
| **Estado** | Vigente |
| **Documentos** | `COACH_PHILOSOPHY.md`, RFC 003 |
| **RFC** | RFC 003 |

---

## D-005 — Race-first en narrativa, no en cambio de programa

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Prioridad narrativa deportiva |
| **Descripción** | En Race, el resultado competitivo manda en la historia. Una victoria no consolida todas las skills ni cambia el foco sola. |
| **Motivación** | El piloto merece reconocimiento deportivo sin sabotear el programa. |
| **Impacto** | Attribution dimensions; Coach abre con resultado en Race/Quali. |
| **Estado** | Vigente |
| **Documentos** | `PERFORMANCE_ATTRIBUTION_ENGINE.md`, `COACH_PHILOSOPHY.md`, Fase B |
| **RFC** | — |

---

## D-006 — Dashboard como centro de mando, no panel de KPIs

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase A |
| **Descripción** | El Dashboard responde en ~30 s: dónde estoy, si mejoro, qué entreno, qué hacer ahora. Pregunta: «¿Qué necesito saber hoy?» |
| **Motivación** | Menos ruido, más decisión. |
| **Impacto** | Eliminación de KPI clutter; objetivo + FocusCallout protagonistas. |
| **Estado** | Vigente · experiencia congelada |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_A.md` |
| **RFC** | — |

---

## D-007 — Coach como briefing continuo de cuatro actos

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase B |
| **Descripción** | Un discurso: qué ocurrió → por qué → significado → qué hacer ahora. Contexto Practice/Quali/Race. |
| **Motivación** | Dejar de listar frases técnicas; hablar como ingeniero de pista. |
| **Impacto** | `coachBriefing.ts`; estructura narrativa fija. |
| **Estado** | Vigente · experiencia congelada |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_B.md`, `NARRATIVE_INTELLIGENCE.md` |
| **RFC** | — |

---

## D-008 — Explainability integrada, sin pantalla nueva

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase C |
| **Descripción** | Sheet lateral «¿Por qué?» con evidencia, confianza, why_not_other y cuándo cambia el foco. Sin IDs/hashes al piloto. |
| **Motivación** | Confianza sin saturar el centro de mando. |
| **Impacto** | `WhyExplanationSheet`; Facts + Strategy proyectados. |
| **Estado** | Vigente · experiencia congelada |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_C.md` |
| **RFC** | — |

---

## D-009 — Trayectoria como biografía deportiva, no lista de IBT

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Driver Experience Fase D |
| **Descripción** | `/driver` narra hitos y etapas (Inicio → … → Nuevo desafío). Sin evidencia → sin hito. |
| **Motivación** | El piloto debe entender «cómo he llegado hasta aquí» en &lt; 2 minutos. |
| **Impacto** | Journey proyecta; no inventa podios ni validaciones. |
| **Estado** | Vigente |
| **Documentos** | `DRIVER_EXPERIENCE_PHASE_D.md` |
| **RFC** | — |

---

## D-010 — Strategy sustituye weaknesses como fuente de foco

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Foco único desde Development Strategy |
| **Descripción** | El Dashboard y el Coach toman el foco de Strategy usable, no de `weaknesses[0]` ni listas de debilidades. |
| **Motivación** | Una sola fuente de verdad de programa; evitar contradicciones. |
| **Impacto** | Priority shift solo si Strategy `action === "change"`. |
| **Estado** | Vigente |
| **Documentos** | `AUDIT_REPORT_RCA1.md`, `DEVELOPMENT_STRATEGY_ENGINE.md` |
| **RFC** | RFC 003 |

---

## D-011 — Lectura HTTP no muta el perfil del piloto

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | GET ≠ escritura (RCA-1) |
| **Descripción** | Abrir Dashboard/Sesiones no reincorpora sesiones. Incorporar es un acto explícito (POST). |
| **Motivación** | «Quitar del perfil» debe persistir; los GET no pueden deshacer decisiones del piloto. |
| **Impacto** | `persist=False` por defecto; `POST …/profile/incorporate`. |
| **Estado** | Vigente |
| **Documentos** | `AUDIT_REPORT_RCA1.md` |
| **RFC** | — |

---

## D-012 — Una historia por piloto activo

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Multi-piloto con piloto activo de producto |
| **Descripción** | Dashboard y Journey filtran por piloto activo seleccionable en Settings. |
| **Motivación** | Evitar biografías mezcladas entre pilotos. |
| **Impacto** | API `GET/PUT /drivers/active`; loaders filtrados. |
| **Estado** | Vigente |
| **Documentos** | `AUDIT_REPORT_RCA1.md` |
| **RFC** | — |

---

## D-013 — Informes fuera del nav en Release Candidate

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Retirar stub de Informes del nav |
| **Descripción** | La entrada Informes generaba expectativa falsa; se retira del nav RC. |
| **Motivación** | Honestidad de producto: no prometer pantallas vacías. |
| **Impacto** | Nav RC sin Informes; reports weekly/career siguen como stubs de Core. |
| **Estado** | Vigente (RC) |
| **Documentos** | `AUDIT_REPORT_RCA1.md` |
| **RFC** | — |

---

## D-014 — DDP 2.0 simplificado: skill → curvas → ejercicio

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Mapa circuito–skill–curvas (v1.1) |
| **Descripción** | Sustituye complejidad de programa semanal/career tables. Planner ancla ejercicios a curvas del circuito. |
| **Motivación** | Entregar valor de entrenamiento sin un segundo producto. |
| **Impacto** | `circuit_skill_map` + YAML; RFC 002. |
| **Estado** | Vigente |
| **Documentos** | `DDP_2_DEVELOPMENT_PROGRAM.md` |
| **RFC** | RFC 002 |

---

## D-015 — Project Memory System como memoria oficial

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | PMS v1 forma parte de la arquitectura |
| **Descripción** | La memoria del proyecto vive en `docs/project/`. Ninguna fase se cierra sin actualizar el PMS. Las conversaciones de IA no son la fuente de verdad. |
| **Motivación** | Supervivencia del conocimiento a 6 meses / 2 años sin depender de chats. |
| **Impacto** | Checklist obligatorio de cierre; orden de lectura de arranque. |
| **Estado** | Vigente |
| **Documentos** | `docs/project/*`, `ARCHITECTURE.md`, `README.md` |
| **RFC** | — |

---

## D-016 — La IA es la voz de DDP, nunca su cerebro

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Papel de la IA en el producto |
| **Descripción** | La IA comunica, explica, enseña, resume, conversa y acompaña. Nunca decide, interpreta telemetría, genera hechos ni modifica Attribution/Strategy. |
| **Motivación** | Preservar determinismo, trazabilidad y confianza. |
| **Impacto** | Arquitectura futura: motores → AI Communication Layer → Conversation → Real Time → Mentor. |
| **Estado** | Vigente |
| **Documentos** | [`AI_GUIDE.md`](AI_GUIDE.md) |
| **RFC** | — |

---

## D-017 — PMS pasa a ser infraestructura oficial (PMI)

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | Project Memory Infrastructure (PMI v1) |
| **Descripción** | El mantenimiento del PMS deja de depender solo de la disciplina humana. Existe infraestructura en `tools/pms/`, especificación `PMS_SPEC.md` y procedimiento oficial `/close-phase`. |
| **Motivación** | Que la memoria del proyecto se mantenga con el mismo rigor que los tests. |
| **Impacto** | Cierre de fase = procedimiento PMI; validación automática; scripts append-only para DECISIONS/ADR. |
| **Estado** | Vigente |
| **Documentos** | `PMS_SPEC.md`, `tools/README.md`, `ARCHITECTURE.md` |
| **RFC** | RFC — Project Memory Infrastructure (PMI v1) |

---

## D-018 — AI Context System como puerta oficial al PMS

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | AICS v1 — entrada diferenciada humanos / IA |
| **Descripción** | Humanos empiezan por `BOOTSTRAP.md`. Las IA empiezan por `AI_BOOTSTRAP.md` y profundizan en el PMS solo cuando hace falta. El AICS no sustituye al PMS. |
| **Motivación** | Reconstruir el contexto del proyecto en &lt; 1 minuto sin depender de chats previos. |
| **Impacto** | `/close-phase` mantiene `AI_BOOTSTRAP` si cambia el contexto; validate_pms exige entradas AICS. |
| **Estado** | Vigente |
| **Documentos** | `BOOTSTRAP.md`, `AI_BOOTSTRAP.md`, `PMS_SPEC.md` |
| **RFC** | RFC — AI Context System (AICS v1) |

---

## D-019 — Configuración global única y prompt ChatGPT generado

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | AICS v2 — PROJECT_CONFIGURATION + ChatGPT Startup Prompt |
| **Descripción** | Toda URL/repo compartido vive solo en `PROJECT_CONFIGURATION.md`. El prompt oficial de ChatGPT se genera automáticamente y vive en `AI_BOOTSTRAP.md`. Existe AI INITIALIZATION PROTOCOL obligatorio. |
| **Motivación** | Incorporar IA sin chats previos; evitar URLs duplicadas y prompts obsoletos. |
| **Impacto** | `/close-phase` regenera el prompt; validate_pms exige sync con la configuración. |
| **Estado** | Vigente |
| **Documentos** | `PROJECT_CONFIGURATION.md`, `AI_BOOTSTRAP.md`, `PMS_SPEC.md` |
| **RFC** | RFC — AI Context System (AICS v2) |

---

## D-020 — Integración GitHub de documentación para ChatGPT

| Campo | Valor |
|-------|-------|
| **Fecha** | 2026-08-08 |
| **Título** | AICS v2.1 — repo público de docs como fuente para ChatGPT |
| **Descripción** | El prompt oficial inyecta Official Documentation Repository, branch y paths desde PROJECT_CONFIGURATION. ChatGPT reconstruye el contexto desde ese repo GitHub. |
| **Motivación** | Que el prompt funcione de verdad con ChatGPT sin editar prompts a mano. |
| **Impacto** | Cambio de URL = solo PROJECT_CONFIGURATION + regenerar prompt; validate_pms prohíbe URLs duplicadas. |
| **Estado** | Vigente |
| **Documentos** | `PROJECT_CONFIGURATION.md`, `AI_BOOTSTRAP.md`, `PMS_SPEC.md` |
| **RFC** | RFC — AICS v2.1 GitHub Documentation Integration |

---

*Añadir nuevas decisiones al final. No borrar las vigentes; marcar estado si se superseden.*


---

# ========== Architecture Decision Records ==========
# Source: docs/project/ADR.md

# DDP — Architecture Decision Records

> Decisiones técnicas importantes.
> No confundir con [`DECISIONS.md`](DECISIONS.md) (producto).
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08

---

## ADR-001 — Core independiente sin HTTP, UI ni SQLite

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | La lógica de dominio no debe morir con el framework de turno. |
| **Decisión** | El Core solo depende de Pandas, NumPy y stdlib. Entrada/salida: estructuras Python puras. Sin efectos (BD, red, UI). |
| **Consecuencias** | Backend es adaptador. Tests de motor sin servidor. Frontend intercambiable. |

---

## ADR-002 — La IA nunca decide

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Modelos generativos opacos rompen trazabilidad y confianza. |
| **Decisión** | Las decisiones de foco, causalidad y hechos las toman motores deterministas. La IA (cuando exista) solo comunica. |
| **Consecuencias** | Misma evidencia → misma conclusión. Ver [`AI_GUIDE.md`](AI_GUIDE.md). |

---

## ADR-003 — La UI nunca interpreta

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Si el Dashboard recalcula causalidad o dimensiones, contradice al Core y al Coach. |
| **Decisión** | Frontend proyecta Attribution y Strategy. Si Attribution falla → estado «insuficiente», no fallback local. |
| **Consecuencias** | Corregido en RCA-1 (A3). Experience layers son proyección. |

---

## ADR-004 — Strategy es la única fuente del foco

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Usar `weaknesses[0]` u otras listas genera focos inconsistentes. |
| **Decisión** | `current_focus` y acción de programa salen de Development Strategy. Priority shift solo si `recommended_action === "change"`. |
| **Consecuencias** | Dashboard/Coach alineados. Planner no redefine el foco. |

---

## ADR-005 — La cronología utiliza `started_at`

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | El orden de importación no es el orden deportivo real. |
| **Decisión** | Toda cronología de perfil/evolución usa `session.started_at`. Import out-of-order exige rebuild. |
| **Consecuencias** | EMA y skills coherentes con la historia real del piloto. |

---

## ADR-006 — Analysis genera hechos de medición

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Mezclar medición y opinión hace imposible auditar números. |
| **Decisión** | Analysis Engine mide (ideal, consistencia, findings). No coachea. No escribe narrativa de programa. |
| **Consecuencias** | Frontera clara con Interpretation y Attribution. Doc `ANALYSIS_ENGINE.md` v1.0 congelada. |

---

## ADR-007 — Attribution interpreta causalidad

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Hace falta explicar *por qué* cambió el rendimiento sin re-medir telemetría. |
| **Decisión** | Performance Attribution consume hechos ya medidos/persistidos y emite `PerformanceAttribution` con dimensiones, confianza y `insufficient_evidence` cuando no hay base. |
| **Consecuencias** | Coach/Dashboard narran Attribution; no la inventan. |

---

## ADR-008 — Planner nunca cambia Strategy

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Si el planner elige el programa, hay dos cerebros. |
| **Decisión** | Planner responde «¿cómo entreno hoy?» a partir del foco de Strategy. No redefine `current_focus`. |
| **Consecuencias** | Cadena Strategy → Planner → Assignment. |

---

## ADR-009 — Interpretation diagnostica sesión, no programa

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Confundir diagnóstico de tanda con foco de desarrollo. |
| **Decisión** | Interpretation Engine prioriza problema de **esta sesión**. El programa multi-sesión es Strategy. |
| **Consecuencias** | Velocidad distinta: interpretación puede cambiar rápido; foco lento. Doc v1.0 congelada. |

---

## ADR-010 — Strategy v1 on-demand sin persistencia

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Persistir Strategy prematuramente acopla esquema a un juicio aún en evolución. |
| **Decisión** | Development Strategy se reconstruye on-demand (RFC 003). Sin tabla Strategy en v1. |
| **Consecuencias** | Siempre refleja Facts/Attribution actuales; sin drift de filas stale. |

---

## ADR-011 — Contrato Core con `inputs_hash` + `engine_version`

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Regeneración y determinismo deben ser verificables. |
| **Decisión** | Motores de import/análisis/interpretación (y Strategy) exponen versión + hash de entradas. Misma evidencia → mismo resultado. |
| **Consecuencias** | Reinterpretar/reanalizar es seguro; regresiones posibles. |

---

## ADR-012 — Persistencia sustituye por política, no acumula duplicados opacos

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Reanálisis / reimportación no deben dejar estados ambiguos. |
| **Decisión** | Analysis: política sustituir (D8). Interpretation: regeneración en el sitio (D5). Import: dedup por `content_hash`. |
| **Consecuencias** | Un resultado coherente por sesión/motor; ver docs de cada motor. |

---

## ADR-013 — Lecturas HTTP no tienen efectos secundarios de perfil

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | GET con `persist=True` reescribía el perfil al abrir pantallas (RCA-1 C1). |
| **Decisión** | Endpoints de lectura de training/skills/progress no persisten por defecto. Escritura solo vía POST incorporate / rebuild / remove. |
| **Consecuencias** | «Quitar del perfil» es estable. |

---

## ADR-014 — Delete de sesión reconstruye cronología antes del hard delete

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Borrar una sesión intermedia contaminaba EMA/skills posteriores. |
| **Decisión** | `delete_session` = remove + rebuild (con `exclude_session_id`) + hard delete. Rebuild vacío limpia career e índices. |
| **Consecuencias** | Historia coherente; deudas M4/M5 documentadas en KNOWN_ISSUES. |

---

## ADR-015 — Project Memory System es parte de la arquitectura

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | El conocimiento vivía en chats y en la cabeza de las personas. |
| **Decisión** | `docs/project/` es memoria obligatoria. Cierre de fase exige checklist PMS. |
| **Consecuencias** | Toda fase futura actualiza STATE, HANDOVER, SESSION_NOTES, etc. |

---

## ADR-016 — Conceptos prohibidos en Strategy

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Modelos con `horizon` / `phase` / `posture` / `competition_policy` sobrecargaban el juicio. |
| **Decisión** | Esos conceptos no se reintroducen en Development Strategy. |
| **Consecuencias** | Modelo simple: focus, reason, why_not_other, state, action, review_condition. |

---

## ADR-017 — Project Memory Infrastructure en tools/pms

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | El PMS documental solo se mantenía por disciplina; era fácil cerrar fases sin actualizar la memoria. |
| **Decisión** | Crear PMI en `tools/pms/` (fuera del runtime). `validate_pms` + `close_phase` son el procedimiento oficial `/close-phase`. DECISIONS/ADR solo crecen (append). `publish_docs` queda reservado para un futuro repo público de documentación. |
| **Consecuencias** | Infra versionada con el repo privado; no toca Core/Frontend/Backend funcional; cierre de fase gateado por validación PMS. |

---

## ADR-018 — AICS: AI_BOOTSTRAP como entrada canónica para IA

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Una IA nueva tenía que leer muchos documentos o depender de historial de chat para entender DDP. |
| **Decisión** | Crear AICS dentro del PMS: `AI_BOOTSTRAP.md` (IA) y `BOOTSTRAP.md` (humanos). PMS sigue siendo la única fuente de verdad. `/close-phase` invoca `update_ai_bootstrap` cuando el contexto general cambia. |
| **Consecuencias** | Reconstrucción &lt; 1 min; menos drift por conversaciones; mantenimiento AICS acoplado al cierre de fase. |

---

## ADR-019 — Una sola fuente de configuración para el AICS

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | URLs y prompts hardcodeados en varios docs se desincronizan y obligan a editar a mano. |
| **Decisión** | `PROJECT_CONFIGURATION.md` es la única fuente de `Official Documentation Repository`. PMI (`update_ai_bootstrap`) inyecta ese valor en el bloque `ChatGPT Startup Prompt` de `AI_BOOTSTRAP.md`. Ningún otro documento escribe la URL manualmente. |
| **Consecuencias** | Cambio de repo = un solo edit + regenerar prompt; validate_pms falla si el prompt está desfasado. |

---

## ADR-020 — Prompt ChatGPT inyecta config GitHub completa

| Campo | Valor |
|-------|-------|
| **Estado** | Aceptada |
| **Problema** | Un prompt con URL fija o incompleta deja de funcionar al mover el repo de documentación. |
| **Decisión** | El generador lee Project Name, Version, Official Documentation Repository, Branch, Docs Path, AI Bootstrap Path y Bootstrap Path desde PROJECT_CONFIGURATION.md e inyecta todos en el bloque ChatGPT Startup Prompt. validate_pms prohíbe duplicar la URL fuera de config + bloque generado. |
| **Consecuencias** | Cambio de repo = un edit en PROJECT_CONFIGURATION + `--refresh-prompt`; ChatGPT siempre recibe la config vigente. |

---

*Nuevos ADR al final. Si se supersede uno, marcar Estado = Superada y enlazar el nuevo.*


---

# ========== Handover ==========
# Source: docs/project/HANDOVER.md

# DDP — Handover

> **Humanos:** [`BOOTSTRAP.md`](BOOTSTRAP.md).  
> **IA:** [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) (+ AI INITIALIZATION PROTOCOL).  
> Config global: [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md).
>
> Solo existe **una versión**. Parte del PMS + AICS v2.

**Última actualización:** 2026-08-08

---

## Última fase terminada

**AICS v2.1 — GitHub Documentation Integration**

- `PROJECT_CONFIGURATION.md` ampliado (nombre, versión, repo, rama, rutas, PMS version).
- ChatGPT Startup Prompt regenerado solo desde esa config (URL + branch + paths).
- `validate_pms`: repo definido · prompt generado · sin URLs duplicadas.
- `/close-phase` regenera el prompt automáticamente.
- Sustituir `TU_USUARIO` en la config cuando el repo público exista.

Anteriores: AICS v2 · PMI v1 · PMS · RCA-1.

---

## Qué se hizo

Infraestructura/documentación únicamente. Sin cambios en motores, backend,
frontend, Strategy, Attribution, Planner ni Coach de producto.

---

## Qué queda pendiente

| Prioridad | Ítem |
|-----------|------|
| **Siguiente gran fase** | Coach en tiempo real / Ingeniero de pista |
| Config | Context Pack + publish docs (`generate_context_pack` → `publish_docs --prepare` → push repo de docs en `PROJECT_CONFIGURATION`) |
| MEDIO | KI-001 / KI-002 |
| BAJO | Resto de KNOWN_ISSUES |

---

## Qué no debe tocarse

Decisiones/ADR congelados · esquema · frontera Core · UI no interpreta ·
Strategy = foco · IA no decide · GET no escribe perfil · append-only DECISIONS/ADR.

No hardcodear URLs de documentación fuera de `PROJECT_CONFIGURATION.md`.

---

## Próximo objetivo

**Coach en tiempo real / Ingeniero de pista.**

IA nueva:

1. Copiar ChatGPT Startup Prompt desde `AI_BOOTSTRAP.md`, **o**
2. Leer `AI_BOOTSTRAP.md` y seguir AI INITIALIZATION PROTOCOL.

```bash
py -m tools.pms.update_ai_bootstrap --refresh-prompt --apply
py -m tools.pms.validate_pms
```

---

## Estado general

| | |
|--|--|
| Producto | DDP 4.0 RC |
| PMS / PMI | Oficiales |
| AICS | **v2.1** |
| Config | `PROJECT_CONFIGURATION.md` (única fuente URL/rama/rutas) |

---

*Una sola versión. Sobrescribir al cerrar cada fase.*


---

# ========== Known Issues ==========
# Source: docs/project/KNOWN_ISSUES.md

# DDP — Known Issues

> Solo incidencias **abiertas**.
> Cuando se resuelva una incidencia → **eliminarla** de este documento.
> No es un changelog. Para historia funcional: [`CHANGELOG_AI.md`](CHANGELOG_AI.md).
>
> Parte del **Project Memory System (PMS v1)**.

**Última revisión:** 2026-08-08  
**Fuente principal:** `docs/AUDIT_REPORT_RCA1.md` § Problemas pendientes

---

## KI-001 — Ficheros `.ibt` huérfanos en disco tras delete

| Campo | Valor |
|-------|-------|
| **Prioridad** | MEDIO |
| **Descripción** | `delete_session` elimina filas de BD pero puede dejar archivos de telemetría en el almacén gestionado. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `backend/importing/service.py` (delete / storage), almacén de telemetría en disco |

---

## KI-002 — Política de `analysis_report` tras borrar sesión

| Campo | Valor |
|-------|-------|
| **Prioridad** | MEDIO |
| **Descripción** | Tras delete, pueden quedar informes de interpretación / narrativa huérfanos o desanclados. Falta decisión de producto: detach vs delete en cascada. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | Repositorios de `analysis_report`, flujo `delete_session` |

---

## KI-003 — Helpers legacy sin callers calientes

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | `evolutionCoach.ts` y `composeCoachBrief.ts` (deprecated) permanecen documentados; no borrar hasta pase de dead-code confirmado. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `frontend/src/pages/dashboard/evolutionCoach.ts`, `composeCoachBrief.ts` |

---

## KI-004 — React Query instalado sin uso real de `useQuery`

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Provider presente; invalidación actual vía event bus `ddp:product-mutated`. Decidir adoptar React Query de verdad o retirar la dependencia. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | Frontend providers / `frontend/src/lib/productSync.ts` |

---

## KI-005 — Stubs Core de reports weekly / career

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Módulos `core/reports/weekly.py` y `career.py` existen como stubs; Informes fuera del nav RC. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `core/reports/weekly.py`, `core/reports/career.py` |

---

## KI-006 — Espejo `sessionContext` TypeScript / Python

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Riesgo de drift entre clasificación de tipo de sesión en Core y tipado/espejo en frontend. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `core/training/session_context.py`, espejo TS en frontend |

---

## KI-007 — Sin endpoint dedicado «eliminar perfil»

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Hoy la limpieza de perfil es parcial vía remove-all / reset DDP. Falta API de producto dedicada. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | `backend/api/training_routes.py`, reset / profile services |

---

## KI-008 — Discard historial importa solo ~200 batches

| Campo | Valor |
|-------|-------|
| **Prioridad** | BAJO |
| **Descripción** | Edge case: el historial de discard/import puede truncar a ~200 batches antiguos. |
| **Fecha** | 2026-08-08 |
| **Estado** | Abierta |
| **Archivos afectados** | Lógica de import log / discard |

---

*Revisar al cerrar cada fase. Resolver → borrar la entrada.*


---

# ========== Roadmap ==========
# Source: docs/project/ROADMAP.md

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


---

# ========== AI Guide ==========
# Source: docs/project/AI_GUIDE.md

# DDP — AI Guide

> Define el papel de la IA dentro de DDP.
> Documento normativo. Ante duda: la IA no manda.
>
> Parte del **Project Memory System (PMS v1)**.

**Última actualización:** 2026-08-08  
**Decisión de producto:** [`DECISIONS.md`](DECISIONS.md) D-016  
**ADR:** [`ADR.md`](ADR.md) ADR-002

---

## Principio absoluto

```text
La IA es la VOZ de DDP.
Nunca su CEREBRO.
```

Los motores deterministas deciden, miden, atribuyen y eligen el foco.
La IA comunica ese conocimiento al piloto.

---

## La IA nunca

| Prohibición | Motivo |
|-------------|--------|
| **Nunca decide** | El foco y las acciones salen de Strategy / política versionada |
| **Nunca interpreta telemetría** | Analysis / Interpretation son motores de hechos y diagnóstico |
| **Nunca modifica Attribution** | Causalidad auditada; no reescribir causas en prosa libre |
| **Nunca modifica Strategy** | El programa no se negocia con un LLM |
| **Nunca genera hechos** | Sin evidencia en Facts/BD/Core → no se afirma |
| **Nunca inventa evidencias** | Confianza = trazabilidad |
| **Nunca elige el foco en la UI** | Proyección solamente |

Si una capacidad propuesta viola esta tabla → **no pertenece a DDP**.

---

## La IA únicamente

| Capacidad | Significado |
|-----------|-------------|
| **Comunica** | Traduce salidas de motor a lenguaje de piloto |
| **Explica** | Ayuda a entender el «por qué» ya calculado |
| **Enseña** | Orienta el aprendizaje sin cambiar el programa |
| **Resume** | Condensa historia y estado sin alterar hechos |
| **Conversa** | Diálogo natural anclado a evidencia |
| **Acompaña** | Presencia continua (visión: tiempo real / mentor) |

La Narrative Intelligence actual del frontend es un precursor **determinista**
de esta capa: ya humaniza sin LLM. Cualquier LLM futuro debe respetar
las mismas fronteras.

---

## Arquitectura futura

```text
Motores deterministas
        ↓
AI Communication Layer
        ↓
Conversation Engine
        ↓
Real Time Engineer
        ↓
Mentor Deportivo
```

| Capa | Rol |
|------|-----|
| **Motores deterministas** | Import, Analysis, Interpretation, Attribution, Strategy Facts, Development Strategy, Planner, Progress, Profile |
| **AI Communication Layer** | Voz: explica y narra salidas ya calculadas |
| **Conversation Engine** | Preguntas del piloto → respuestas ancladas a Facts/Strategy/Attribution |
| **Real Time Engineer** | Acompañamiento durante la sesión (tiempo real) |
| **Mentor Deportivo** | Arco a largo plazo; compañero de carrera |

Ninguna capa superior puede saltarse los motores para “decidir mejor”.

---

## Contrato con el resto del sistema

```text
Hechos + Attribution + Strategy + Planner
              │
              ▼
     AI Communication Layer   ← solo lectura de salidas
              │
              ▼
        Piloto / UI
```

- Entrada de la IA: estructuras ya producidas por el Core/Backend.
- Salida de la IA: texto / voz / diálogo.
- Efectos prohibidos: escribir Strategy, Attribution, Facts, Analysis, perfil.

---

## Relación con el Coach actual

Hoy el Coach de superficie (Dashboard briefing, Explainability, Journey)
es **proyección determinista** + Narrative Intelligence.

Eso es correcto y permanece válido aunque más adelante exista un LLM:

1. Primero los motores producen la verdad.
2. Después la capa de comunicación la dice bien.
3. Nunca al revés.

Ver: `COACH_PHILOSOPHY.md`, Fases B–D, `NARRATIVE_INTELLIGENCE.md`.

---

## Criterio de aceptación para cualquier feature de IA

Antes de implementar:

1. ¿Qué motor produce el hecho o la decisión?
2. ¿La IA solo lo comunica?
3. ¿Puede el piloto pedir el «por qué» anclado a evidencia?
4. ¿Misma evidencia → misma decisión de motor (aunque cambie el fraseo)?

Si (2) es no → rediseñar.

---

## Visión alineada

Coach en tiempo real → Ingeniero de pista → IA conversacional → Mentor deportivo.

Detalle de producto: [`ROADMAP.md`](ROADMAP.md).  
Filosofía: [`DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md).

---

*La IA representa la voz de DDP. Nunca su cerebro.*


---

# ========== Human Bootstrap ==========
# Source: docs/project/BOOTSTRAP.md

# DDP — Bootstrap (humanos)

> Punto oficial de entrada para cualquier **desarrollador humano**.
>
> Las Inteligencias Artificiales deben empezar por
> [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) (AI Context System).
>
> Parte del **Project Memory System (PMS)** + **AICS v2.1**.

**Última actualización:** 2026-08-08

---

## Qué es el PMS

El **Project Memory System** es la memoria oficial de DDP.

Vive en `docs/project/`. No es documentación opcional.

Es la **única fuente oficial de verdad** del estado, las decisiones y el
historial del proyecto. Los chats (Cursor, ChatGPT u otros) **no** son
fuente de verdad.

La infraestructura que lo valida y cierra fases es la **PMI**
(`tools/pms/`, procedimiento `/close-phase`).

Especificación: [`PMS_SPEC.md`](PMS_SPEC.md).  
**Única fuente válida** de URL del repositorio, rama, rutas oficiales y
configuración AICS: [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md).
Nunca escribir esa URL a mano en otros documentos.

---

## AI Context System (AICS)

| Entrada | Para quién |
|---------|------------|
| **Este documento** (`BOOTSTRAP.md`) | Humanos |
| [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) | Inteligencias Artificiales |

El AICS **no sustituye** al PMS. Es la **puerta de entrada** al PMS.

Incluye:

- **AI INITIALIZATION PROTOCOL** (obligatorio para toda IA)
- **ChatGPT Startup Prompt** (prompt oficial generado desde
  `PROJECT_CONFIGURATION.md` — nunca editar la URL a mano)

---

## Documentos del PMS

| Documento | Propósito | Cuándo leerlo |
|-----------|-----------|---------------|
| [`BOOTSTRAP.md`](BOOTSTRAP.md) | Entrada humana al PMS | Al incorporar un desarrollador |
| [`AI_BOOTSTRAP.md`](AI_BOOTSTRAP.md) | Entrada IA + protocolo + prompt ChatGPT | Al iniciar cualquier conversación con IA |
| [`PROJECT_CONFIGURATION.md`](PROJECT_CONFIGURATION.md) | Configuración global (única fuente de URLs/repos) | Al cambiar repo de docs o regenerar prompts |
| [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) | Incorporación &lt; 5 min | Tras filosofía/arquitectura |
| [`PROJECT_STATE.md`](PROJECT_STATE.md) | Estado actual visual | Siempre antes de desarrollar |
| [`DECISIONS.md`](DECISIONS.md) | Decisiones de producto | Antes de cambiar comportamiento de producto |
| [`ADR.md`](ADR.md) | Decisiones técnicas | Antes de tocar arquitectura/motores/contratos |
| [`ROADMAP.md`](ROADMAP.md) | Evolución de producto | Para ubicar “qué sigue” |
| [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md) | Incidencias abiertas | Antes de tocar deuda o delete/perfil |
| [`HANDOVER.md`](HANDOVER.md) | Relievo de fase | Al retomar trabajo / nueva sesión |
| [`SESSION_NOTES.md`](SESSION_NOTES.md) | Diario cronológico | Para historia de fases |
| [`CHANGELOG_AI.md`](CHANGELOG_AI.md) | Cambios funcionales | Para entender el producto percibido |
| [`AI_GUIDE.md`](AI_GUIDE.md) | Papel de la IA | Antes de proponer features de IA |
| [`PMS_SPEC.md`](PMS_SPEC.md) | Spec PMS/PMI/AICS | Al cambiar infra de memoria |
| [`PHASE_CLOSE_CHECKLIST.md`](PHASE_CLOSE_CHECKLIST.md) | Checklist de cierre | Al cerrar una fase |

Norte de producto (fuera de `project/` pero obligatorio):

| Documento | Propósito |
|-----------|-----------|
| [`../DDP_PHILOSOPHY.md`](../DDP_PHILOSOPHY.md) | Qué es DDP y qué merece existir |
| [`../ARCHITECTURE.md`](../ARCHITECTURE.md) | Capas, pipeline, contrato Core |

---

## Orden oficial de lectura

Antes de desarrollo importante:

1. `docs/DDP_PHILOSOPHY.md`
2. `docs/ARCHITECTURE.md`
3. `docs/project/PROJECT_CONTEXT.md`
4. `docs/project/PROJECT_STATE.md`
5. `docs/project/DECISIONS.md`
6. `docs/project/ADR.md`
7. `docs/project/HANDOVER.md`
8. `docs/project/KNOWN_ISSUES.md`

**Atajo para humanos nuevos:** este `BOOTSTRAP.md` → luego el orden anterior.  
**Atajo para IA:** `AI_BOOTSTRAP.md` → profundizar solo con enlaces del PMS.

---

## Prioridad en caso de contradicción

```text
DDP_PHILOSOPHY
      ↓
ARCHITECTURE
      ↓
ADR
      ↓
DECISIONS
      ↓
PROJECT_STATE
      ↓
HANDOVER
```

1. La **misión** manda sobre la implementación.
2. La **arquitectura** manda sobre atajos de capa.
3. Los **ADR** mandan sobre detalles técnicos ad hoc.
4. Las **DECISIONS** de producto mandan sobre preferencias locales.
5. **STATE** y **HANDOVER** describen el presente operativo; si chocan con
   ADR/DECISIONS/filosofía, se corrigen STATE/HANDOVER (no al revés).

Los chats nunca ganan a esta cadena.

---

## Consulta rápida según necesidad

| Necesitas… | Consulta |
|------------|----------|
| Entender qué es DDP | `DDP_PHILOSOPHY.md` |
| Ver capas y pipeline | `ARCHITECTURE.md` |
| Estado “ahora” | `PROJECT_STATE.md` + `HANDOVER.md` |
| Si puedo tocar un motor | `ADR.md` + docs congelados del motor |
| Por qué el foco es X | `DECISIONS.md` + Strategy docs |
| Qué está roto | `KNOWN_ISSUES.md` |
| Papel de la IA | `AI_GUIDE.md` |
| Cerrar una fase | `/close-phase` · `PMS_SPEC.md` · checklist |

---

## Cierre de fase

```bash
py -m tools.pms.validate_pms
py -m tools.pms.close_phase --phase "Nombre"
```

Tras cambios de contexto general, `/close-phase` regenera
`AI_BOOTSTRAP.md` y el **ChatGPT Startup Prompt** desde
`PROJECT_CONFIGURATION.md`.

`BOOTSTRAP.md` solo cambia si cambia la **estructura** del PMS.  
Ninguna URL de documentación se duplica fuera de `PROJECT_CONFIGURATION.md`.

---

## Regla de oro

> El conocimiento vive en el proyecto. Nunca en los chats.

El PMS es la única fuente oficial de verdad.

---

*AICS v2.1 · puerta humana al PMS.*


---

# ========== PMS Specification ==========
# Source: docs/project/PMS_SPEC.md

# DDP — Project Memory System Specification

> **Especificación oficial del PMS y de la Project Memory Infrastructure (PMI v1).**
>
> Este documento define qué es el PMS, qué documentos existen, quién puede
> modificarlos, cuándo se actualizan, qué formato tienen, qué validaciones
> pasan, qué scripts los mantienen y cómo se relacionan entre sí.
>
> Ante conflicto operativo de memoria del proyecto, manda este documento
> junto con `HANDOVER.md` / `PROJECT_STATE.md`.
> Ante conflicto de *misión de producto*, manda `DDP_PHILOSOPHY.md`.

**Versión:** PMI v1 + AICS v2.1 · 2026-08-08  
**Estado:** Vigente — infraestructura oficial de desarrollo  
**Comando de cierre:** `/close-phase`

---

## 1. Qué es el PMS

El **Project Memory System** es la memoria oficial de DDP.

Vive en `docs/project/`. No es documentación opcional ni un conjunto de notas.
Es la **única fuente oficial de verdad** del proyecto. Los chats no lo son.

La **Project Memory Infrastructure (PMI)** es la capa de herramientas en
`tools/pms/` que valida, actualiza y orquesta el mantenimiento del PMS.

El **AI Context System (AICS)** es la puerta de entrada al PMS:

| Entrada | Audiencia |
|---------|-----------|
| `BOOTSTRAP.md` | Desarrolladores humanos |
| `AI_BOOTSTRAP.md` | Inteligencias Artificiales |

El AICS **no sustituye** al PMS. Resume, enlaza, define el
**AI INITIALIZATION PROTOCOL** y genera el **ChatGPT Startup Prompt**.

```text
Development Infrastructure
        ↓
Project Memory System (PMS)
        ↓
AI Context System (AICS)
        ↓
Product Architecture
```

```text
docs/project/     →  PMS + AICS + PROJECT_CONFIGURATION
tools/pms/        →  PMI (scripts)
```

El producto runtime (`core/`, `backend/`, `frontend/`) **no** incluye la PMI.

---

## 1.1 PROJECT_CONFIGURATION.md

**Única fuente de configuración global** del AICS / PMS / PMI.

| Campo | Uso |
|-------|-----|
| `Project Name` | Nombre del proyecto |
| `Current Version` | Versión de producto |
| `Official Documentation Repository` | URL GitHub del repo público de documentación |
| `Repository Branch` | Rama (p. ej. `main`) |
| `Repository Docs Path` | Raíz de docs en ese repo |
| `AI Bootstrap Path` | Ruta a `AI_BOOTSTRAP.md` |
| `Bootstrap Path` | Ruta a `BOOTSTRAP.md` |
| `PMS Version` | Versión del sistema de memoria / AICS |

**Reglas:**

- Ningún otro documento escribe la URL (ni rama/rutas) a mano.
- El **ChatGPT Startup Prompt** se genera solo desde estos campos.
- Si cambia la URL → editar **solo** este archivo → regenerar prompt
  (`update_ai_bootstrap --refresh-prompt --apply` o `/close-phase`).
- `validate_pms` falla si la URL aparece duplicada fuera de este archivo
  y del bloque generado del prompt.

Mantenedor: humano (valores) + PMI (propagación al prompt).

---

## 1.2 AICS — BOOTSTRAP / AI_BOOTSTRAP

| Documento | Quién lo mantiene | Cuándo |
|-----------|-------------------|--------|
| `BOOTSTRAP.md` | Humano / agente | Solo si cambia la **estructura** del PMS |
| `AI_BOOTSTRAP.md` | Humano / agente + PMI | Si cambia contexto general; prompt siempre regenerable |

### AI INITIALIZATION PROTOCOL

Sección obligatoria dentro de `AI_BOOTSTRAP.md`. Define el procedimiento
que toda IA debe seguir antes de desarrollar (lectura de bootstrap, obtención
del repo desde `PROJECT_CONFIGURATION.md`, lectura PMS obligatoria, PMS &gt; chat).

### ChatGPT Startup Prompt

Sección obligatoria en `AI_BOOTSTRAP.md`, delimitada por:

```text
<!-- AICS:CHATGPT_PROMPT_BEGIN -->
…
<!-- AICS:CHATGPT_PROMPT_END -->
```

- **No** editar a mano el bloque.
- Generado por `tools/pms/update_ai_bootstrap.py` leyendo
  `PROJECT_CONFIGURATION.md`.
- Es el prompt oficial copy-paste para ChatGPT.

---

## 2. Documentos del PMS

| Documento | Propósito | Mutabilidad |
|-----------|-----------|-------------|
| `BOOTSTRAP.md` | Entrada humana (AICS) | Solo si cambia la **estructura** del PMS |
| `AI_BOOTSTRAP.md` | Entrada IA + protocolo + prompt ChatGPT | Contexto general + prompt auto |
| `PROJECT_CONFIGURATION.md` | Config global (repos/URLs) | Cuando cambie el repo oficial de docs |
| `PROJECT_CONTEXT.md` | Incorporación (&lt; 5 min) | Actualizar cuando cambie la visión/arquitectura general |
| `PROJECT_STATE.md` | Estado actual visual | **Cada cierre de fase** |
| `DECISIONS.md` | Decisiones de producto | **Append-only** (nunca reescribir vigentes) |
| `ADR.md` | Decisiones técnicas | **Append-only** (nunca reescribir aceptadas) |
| `ROADMAP.md` | Evolución de producto | Actualizar si cambia lo conseguido/siguiente/visión |
| `KNOWN_ISSUES.md` | Solo issues abiertos | Añadir / **eliminar al resolver** |
| `HANDOVER.md` | Relievo (una sola versión) | **Sobrescribir** en cada cierre |
| `SESSION_NOTES.md` | Diario cronológico | **Append-only** |
| `CHANGELOG_AI.md` | Cambios funcionales | **Append-only** (entradas nuevas) |
| `AI_GUIDE.md` | Papel de la IA | Actualizar si cambia la norma de IA |
| `PMS_SPEC.md` | Esta especificación | Actualizar cuando cambie la infra PMS/PMI/AICS |
| `PHASE_CLOSE_CHECKLIST.md` | Checklist reutilizable | Plantilla; copias por fase opcionales |

Documentos obligatorios incluyen siempre las puertas AICS + el núcleo PMS.

---

## 3. Quién puede modificarlos

| Actor | Permisos |
|-------|----------|
| Desarrollador / agente en fase | Actualizar PMS como parte del cierre |
| PMI (`tools/pms/*`) | Validar; append seguro; stamp de fechas; listar issues |
| Procedimientos futuros CI | Ejecutar `validate_pms` / `/close-phase` |
| Nadie | Reescribir en silencio decisiones/ADR congelados |
| Nadie | Usar chats como fuente de verdad frente al PMS |

**Regla:** DECISIONS y ADR solo crecen. Si una decisión se supersede, se marca
su `Estado` y se añade una nueva entrada — no se borra la historia.

---

## 4. Cuándo se actualizan

| Momento | Documentos |
|---------|------------|
| Durante desarrollo | Opcional: SESSION_NOTES parciales, issues nuevos |
| Antes de cerrar fase | STATE, HANDOVER, SESSION_NOTES (obligatorio) |
| Si hay decisión de producto | DECISIONS (nueva entrada) |
| Si hay decisión técnica | ADR (nueva entrada) |
| Si cambia el producto percibido | CHANGELOG_AI, ROADMAP |
| Si se abre/cierra incidencia | KNOWN_ISSUES |
| Si cambia contexto general | **`AI_BOOTSTRAP.md`** (filosofía, arquitectura, estado, ADR, decisiones, roadmap, próxima fase, issues críticos) + regenerar **ChatGPT Startup Prompt** |
| Si cambia `Official Documentation Repository` | Solo `PROJECT_CONFIGURATION.md` → regenerar prompt |
| Si cambia estructura del PMS | **`BOOTSTRAP.md`** |
| Tras `/close-phase` OK | Fase puede declararse CERRADA |

Sin actualización PMS → fase **ABIERTA**, aunque el código y los tests pasen.

---

## 5. Formatos

### DECISIONS
Cada entrada `## D-NNN — Título` con campos: Fecha, Título, Descripción,
Motivación, Impacto, Estado, Documentos, RFC.

### ADR
Cada entrada `## ADR-NNN — Título` con campos: Estado, Problema, Decisión,
Consecuencias.

### KNOWN_ISSUES
Solo abiertas. Entrada `## KI-NNN — Título` con Prioridad, Descripción,
Fecha, Estado, Archivos afectados. Al resolver → **eliminar** la entrada.

### SESSION_NOTES
Entradas `## YYYY-MM-DD — Título` con Objetivo, Trabajo, Problemas,
Decisiones, Resultado. Nunca sobrescribir historial.

### HANDOVER
Una sola versión viva. Secciones mínimas: última fase, qué se hizo, pendiente,
qué no tocar, próximo objetivo, estado general.

### PROJECT_STATE
Debe incluir versión, estado, motores, frontend/backend, última actualización
(fecha ISO).

---

## 6. Validaciones (`validate_pms`)

`py -m tools.pms.validate_pms` comprueba automáticamente:

| Check | Criterio |
|-------|----------|
| Documentos obligatorios | Existen en `docs/project/` |
| No vacíos | Documentos core con contenido real |
| Enlaces | Links relativos resolubles |
| PROJECT_STATE | Marcadores de estado + fecha |
| HANDOVER | Secciones mínimas |
| SESSION_NOTES | Al menos una entrada fechada |
| KNOWN_ISSUES | Formato válido |
| DECISIONS | Formato D-NNN + campos |
| ADR | Formato ADR-NNN + campos |
| PMS_SPEC | Presente y referencia cierre |

Exit code `0` = PASS · `1` = FAIL (fase abierta).

---

## 7. Scripts PMI (`tools/pms/`)

| Script | Responsabilidad |
|--------|-----------------|
| `close_phase.py` | Orquesta `/close-phase` (incluye AICS) |
| `validate_pms.py` | Informe de validación (incluye AICS) |
| `sync_docs.py` | Sync privado `docs/`; sonda repo público futuro |
| `publish_docs.py` | **Reservado** — publicación pública |
| `generate_handover.py` | Apoyo a generación de HANDOVER |
| `update_project_state.py` | Stamp / apoyo STATE |
| `update_changelog.py` | Append CHANGELOG_AI |
| `update_known_issues.py` | List / add / remove issues |
| `update_decisions.py` | Append decisiones (nunca reescribe) |
| `update_adr.py` | Append ADR (nunca reescribe) |
| `update_ai_bootstrap.py` | Mantener `AI_BOOTSTRAP.md` + regenerar ChatGPT Startup Prompt |
| `project_config.py` | Leer `PROJECT_CONFIGURATION.md` (única fuente de repo de docs) |
| `utils.py` | Rutas, constantes, reportes |

Detalle de uso: [`tools/README.md`](../../tools/README.md).

---

## 8. Relaciones entre documentos

```text
BOOTSTRAP (humanos) ──┐
AI_BOOTSTRAP (IA)   ──┼──► PMS (fuente de verdad)
                      │
DDP_PHILOSOPHY ──norte──► producto
ARCHITECTURE ──capas──► runtime + PMS/PMI/AICS
        │
        ▼
PROJECT_CONTEXT ──incorpora──► STATE / DECISIONS / ADR / HANDOVER
PROJECT_STATE   ◄──espejo──► HANDOVER (estado actual)
DECISIONS       ◄──producto──► ROADMAP / CHANGELOG_AI
ADR             ◄──técnico──► ARCHITECTURE / motores
KNOWN_ISSUES    ◄──deuda──► HANDOVER pendiente
SESSION_NOTES   ──diario──► (append; alimenta memoria)
AI_GUIDE        ──norma IA──► DECISIONS / ADR
PMS_SPEC        ──norma PMS──► tools/pms/*
```

### Entradas oficiales

| Quién | Empieza por |
|-------|-------------|
| Humano | `BOOTSTRAP.md` |
| IA | `AI_BOOTSTRAP.md` |

Orden de lectura profundo (desarrollo importante):

1. `DDP_PHILOSOPHY.md`
2. `ARCHITECTURE.md`
3. `PROJECT_CONTEXT.md`
4. `PROJECT_STATE.md`
5. `DECISIONS.md`
6. `ADR.md`
7. `HANDOVER.md`
8. `KNOWN_ISSUES.md`

Prioridad ante contradicción:

```text
PHILOSOPHY → ARCHITECTURE → ADR → DECISIONS → STATE → HANDOVER
```

---

## 9. Procedimiento oficial `/close-phase`

Concepto oficial de DDP (automatizable; no requiere slash-command del editor):

```text
1.  Desarrollar
2.  Ejecutar tests
3.  Ejecutar validación PMS          → validate_pms
4.  Actualizar documentación PMS
5.  Generar HANDOVER                 → generate_handover
6.  Revisar Issues                   → update_known_issues
7.  Actualizar PROJECT_STATE         → update_project_state
8.  Actualizar CHANGELOG             → update_changelog
9.  Actualizar ADR / DECISIONS       → update_adr / update_decisions
10. Mantener AICS                    → update_ai_bootstrap
      (AI_BOOTSTRAP si cambió contexto;
       ChatGPT Startup Prompt siempre regenerable desde PROJECT_CONFIGURATION)
11. Sincronizar documentación        → sync_docs
12. Validación final + Cerrar fase   → validate_pms PASS
```

Comando:

```bash
py -m tools.pms.close_phase --phase "Nombre de la fase"
```

Si cualquier paso falla → la fase permanece **ABIERTA**.

Checklist: [`PHASE_CLOSE_CHECKLIST.md`](PHASE_CLOSE_CHECKLIST.md).

---

## 10. Repositorio público de documentación (futuro)

Arquitectura preparada; **no implementada** en PMI v1:

```text
Repositorio privado (código + docs/)
        ↓  sync_docs (selección)
        ↓  publish_docs (publicación)
Repositorio público (solo documentación)
```

- Variable de entorno prevista: `DDP_PUBLIC_DOCS_REPO`
- Placeholder de ruta: `../ddp-docs`
- `publish_docs.py` está reservado y no publica todavía
- Ningún script debe asumir que el código de producto se publica

---

## 11. Integración con Git

| Práctica | Norma |
|----------|-------|
| Fuente de verdad | Repositorio privado |
| PMS en Git | Sí — `docs/project/` versionado |
| PMI en Git | Sí — `tools/pms/` versionado |
| Cierre de fase | Commit(s) que incluyen código **y** PMS actualizado |
| Hooks / CI futuros | Podrán ejecutar `validate_pms` como gate |

PMI v1 no instala hooks automáticamente; deja la puerta abierta.

---

## 12. Restricciones

La PMI / PMS **no**:

- modifica motores del Core
- modifica frontend de producto
- modifica backend funcional
- cambia la arquitectura runtime de DDP

Solo infraestructura de desarrollo y memoria del proyecto.

---

## 13. Confirmación normativa

A partir de PMI v1 + AICS v2.1:

1. El PMS es **infraestructura oficial** de DDP.
2. El AICS es la **puerta oficial** al PMS (humanos / IA).
3. `PROJECT_CONFIGURATION.md` es la **única fuente** de URL, rama y rutas AICS.
4. El **ChatGPT Startup Prompt** nunca se edita a mano; siempre se regenera.
5. ChatGPT usa el repositorio GitHub de documentación definido en la config.
6. El cierre de fase (`/close-phase`) regenera el prompt automáticamente.
7. El conocimiento vive en DDP, nunca en los chats.

---

*PMS_SPEC · PMI v1 · AICS v2.1 · referencia oficial.*


---

