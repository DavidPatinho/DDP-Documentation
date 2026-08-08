# ChatGPT Context Pack

ChatGPT often **cannot browse GitHub**. Use this pack instead.

## Files

| File | Use |
|------|-----|
| `DDP_AICS_CONTEXT.md` | **Recommended** — single file to upload to ChatGPT |
| `DDP_AICS_CONTEXT.zip` | Same content as ZIP (bundle + individual docs) |

## Download links (public docs repo)

- ZIP: https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.zip
- Markdown: https://github.com/DavidPatinho/DDP-Documentation/raw/main/docs/chatgpt/DDP_AICS_CONTEXT.md

## How to use with ChatGPT

1. Download the ZIP or the `.md` file (links above), **or** upload the local file from this folder.
2. In a new ChatGPT chat, **attach/upload** the file.
3. Paste the official **ChatGPT Startup Prompt** from `docs/project/AI_BOOTSTRAP.md`.
4. ChatGPT must read the pack, run AI INITIALIZATION PROTOCOL, reconstruct context, and wait.

Regenerate this pack from the private DDP repo:

```bash
py -m tools.pms.generate_context_pack --apply
py -m tools.pms.publish_docs --prepare
# then push the public DDP-Documentation repo
```
