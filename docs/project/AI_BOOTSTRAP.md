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
