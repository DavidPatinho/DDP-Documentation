# DDP — Project Configuration

> **Única fuente de configuración global** del PMS / AICS / PMI.
>
> Ningún otro documento debe duplicar estos valores (URLs, rama, rutas).
> Si cambia el repositorio de documentación, **solo se edita este archivo**;
> después se regenera el ChatGPT Startup Prompt:
>
> ```bash
> py -m tools.pms.update_ai_bootstrap --refresh-prompt --apply
> ```
>
> También se regenera automáticamente desde `/close-phase`.

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

## Rules

1. **Single source:** URL, rama y rutas oficiales solo aquí.
2. **No duplication:** ningún otro documento escribe la URL a mano.
3. **Regeneration:** cambiar este archivo ⇒ regenerar ChatGPT Startup Prompt.
4. **ChatGPT** usa este repositorio público como fuente de documentación.
5. Publicar solo documentación (sin código de producto). Flujo:
   `py -m tools.pms.publish_docs --prepare`

---

*Configuración global · no es narrativa de producto.*
