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
https://raw.githubusercontent.com/DavidPatinho/DDP-Documentation/main/docs/chatgpt/DDP_AICS_CONTEXT.zip

Context Pack Markdown Download URL:
https://raw.githubusercontent.com/DavidPatinho/DDP-Documentation/main/docs/chatgpt/DDP_AICS_CONTEXT.md

---

## Rules

1. **Single source:** URL, rama, rutas y enlaces de pack solo aquí.
2. **No duplication:** ningún otro documento escribe esas URLs a mano (excepto el prompt generado).
3. Regenerar pack + prompt tras cambios de contexto:
   `generate_context_pack` → `update_ai_bootstrap` → `publish_docs --prepare` → push docs repo.
4. Para ChatGPT: el usuario **descarga/sube** el context pack; no hace falta que ChatGPT entre en GitHub.

---

*Configuración global · no es narrativa de producto.*
