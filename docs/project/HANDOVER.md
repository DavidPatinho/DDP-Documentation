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
| Config | Publicar `https://github.com/DavidPatinho/DDP-Documentation` (`py -m tools.pms.publish_docs --prepare` + push) |
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
