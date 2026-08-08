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
