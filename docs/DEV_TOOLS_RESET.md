# DDP 4.0 — Herramientas de desarrollo (Reinicio)

Fecha: 2026-08-08  
Alcance: mantenimiento local. No forma parte de la experiencia piloto.

**No modificado:** Analysis · Attribution · Strategy Facts · Development Strategy · Training Planner · Dashboard · Coach (motores / experiencias).

---

## Objetivo

Permitir el ciclo continuo de pruebas:

importar → comprobar → **Reiniciar DDP** → importar de nuevo  

sin borrar `ddp.db`, sin scripts externos y sin cerrar la aplicación.

---

## Archivos creados

| Archivo | Rol |
|---------|-----|
| `backend/maintenance/__init__.py` | Paquete |
| `backend/maintenance/service.py` | Servicio de estado + reinicio |
| `backend/api/maintenance_routes.py` | `GET /dev/status`, `POST /dev/reset` |
| `docs/DEV_TOOLS_RESET.md` | Este informe |

## Archivos modificados

| Archivo | Cambio |
|---------|--------|
| `backend/deps.py` | `get_maintenance_service()` |
| `backend/main.py` | Router de mantenimiento |
| `frontend/src/services/api.ts` | `getDevStatus` / `resetDdp` |
| `frontend/src/pages/Settings.tsx` | Sección Herramientas de desarrollo |
| `frontend/src/theme/navigation.ts` | Subtítulo Configuración |

## Componentes UI añadidos

- Resumen **Estado de DDP** (conteos en vivo)
- Zona destructiva con botón **Reiniciar DDP**
- Diálogo de confirmación (`alertdialog`) con checkbox obligatorio + Cancelar / Reiniciar DDP

---

## Servicio de reinicio

`DevelopmentMaintenanceService`

- `status()` → conteos reales (pilotos, telemetrías, sesiones por tipo, vueltas, análisis, objetivos activos…)
- `reset(confirm="REINICIAR")` → vaciado completo del contenido de usuario

### API

```
GET  /dev/status
POST /dev/reset   body: { "confirm": "REINICIAR" }
```

Sin la palabra exacta `REINICIAR`, el reinicio no se ejecuta.

---

## Entidades / tablas limpiadas

En una sola transacción SQLite:

1. Rompe RESTRICTs (`objective_progress`, `objective.supersedes_id`, `milestone.session_id`, `track_profile`)
2. `analysis_report_*` + `analysis_report`
3. `driver_index_dimension` / `driver_index`
4. `career`
5. `track_profile_corner` / `track_profile`
6. `analysis_corner_finding` / `analysis`
7. `objective_progress` / `objective` / `milestone`
8. `session` (CASCADE: laps, metrics, telemetry_file, external ids)
9. `import_batch`
10. `driver` (CASCADE: `driver_external_id`)
11. Claves de `app_setting` ligadas a usuario: `active_driver_id`, `current_car_id`, `current_season_id`, `dashboard_sessions`

Fuera de la TX:

12. Ficheros bajo `database/telemetry/`

### Conservado

- `schema_migration`
- Catálogos (`track`, `car`, `corner`, `series`, `season`, …)
- Preferencias no ligadas a usuario (tema, idioma, rutas de importación…)
- El fichero `database/ddp.db` (no se borra)

---

## Flujo completo

1. Usuario abre **Configuración → Herramientas de desarrollo**
2. Ve el **Estado de DDP** actualizado
3. Pulsa **Reiniciar DDP**
4. Diálogo de confirmación + checkbox «Entiendo…»
5. Confirma → `POST /dev/reset` con `{ confirm: "REINICIAR" }`
6. Backend vacía tablas + telemetrías en disco
7. UI refresca el estado (`is_empty: true`)
8. Dashboard / Trayectoria / Sesiones quedan vacíos sin reiniciar la app
9. Listo para importar nuevas telemetrías desde Telemetría

---

## Verificación

Probado en copia temporal de `ddp.db`:

- Antes: 7 sesiones, 3 pilotos, 10 telemetrías  
- Después: todo a 0, `is_empty: true`  
- Confirmación incorrecta → `ok: false`, sin cambios  

---

## Confirmación de producto

Tras el reinicio, DDP queda igual que una instalación nueva en cuanto a **contenido de usuario**: sin piloto, sin historial, sin trayectoria, sin objetivos, sin análisis, sin Attribution/Strategy/Facts/Planner persistidos, sin sesiones ni telemetrías — y listo para importar de inmediato.
