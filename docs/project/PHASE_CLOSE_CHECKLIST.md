# DDP — Phase Close Checklist

> Checklist reutilizable. Si alguna casilla queda sin marcar,
> la fase permanece **ABIERTA**.
>
> Procedimiento oficial: **`/close-phase`**
> (`py -m tools.pms.close_phase --phase "..."`).
>
> Especificación: [`PMS_SPEC.md`](PMS_SPEC.md).

**Fase:** ______________________________  
**Fecha:** ______________________________  
**Responsable:** ______________________________

---

## Checklist obligatorio

- [ ] Código compila / arranca
- [ ] Tests OK (`pytest`)
- [ ] Arquitectura respetada (`ARCHITECTURE.md`)
- [ ] Filosofía respetada (`DDP_PHILOSOPHY.md`)
- [ ] PMS actualizado
- [ ] Handover actualizado (`HANDOVER.md`)
- [ ] Session Notes actualizado (`SESSION_NOTES.md` — nueva entrada)
- [ ] State actualizado (`PROJECT_STATE.md`)
- [ ] Changelog actualizado (`CHANGELOG_AI.md` — si hubo cambio funcional)
- [ ] Issues revisados (`KNOWN_ISSUES.md`)
- [ ] ADR revisados (`ADR.md` — nuevos si proceden; no reescribir congelados)
- [ ] Decisiones revisadas (`DECISIONS.md` — nuevas si proceden; no reescribir congeladas)
- [ ] AICS revisado (`AI_BOOTSTRAP.md` si cambió contexto; ChatGPT Startup Prompt regenerado desde `PROJECT_CONFIGURATION.md`)
- [ ] `validate_pms` PASS
- [ ] `sync_docs` OK (docs privadas)

---

## Comando

```bash
py -m tools.pms.close_phase --phase "Nombre de la fase"
```

Dry-run:

```bash
py -m tools.pms.close_phase --phase "Nombre de la fase" --dry-run --skip-tests
```

---

## Resultado

- [ ] FASE CERRADA
- [ ] FASE ABIERTA (motivo): ______________________________

---

*PMI v1 · infraestructura oficial de desarrollo.*
