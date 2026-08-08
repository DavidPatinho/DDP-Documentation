# DDP UI 1.0 — Layout Global checklist (Fase 3)

## Shell

- [x] Sidebar + header fijos en viewport
- [x] Único scroll vertical en `main-scroll`
- [x] Sin scroll en `body` / `#root`
- [x] Sin doble scrollbar
- [x] `overflow-x-hidden` en scrollport
- [x] Tablas largas: scroll interno opcional

## Ritmo

- [x] Zonas: 48px (`gap-12`)
- [x] Cards: 24px (`gap-6`)
- [x] Componentes: 16px (`gap-4`)
- [x] Label → valor: 8px (`gap-2`)

## Header / Sidebar

- [x] Header: título, subtítulo, estado, slot acciones
- [x] Header discreto (h-14, borde suave)
- [x] Sidebar: hover / active (barra primary) / focus ring / collapsed

## Cards / Focus

- [x] Bordes suaves (`border-border/50`) — sin borde verde grueso
- [x] FocusCallout más protagonista que HeroCard
- [x] Más padding / tipo / CTA en FocusCallout

## Primer viewport (UI Gallery)

- [x] FocusCallout + Hero + KPIs en la parte superior
- [x] Técnico más abajo

## Pantallas (scroll only — sin rediseño de negocio)

- [x] Dashboard
- [x] Sesiones / detalle
- [x] Telemetría
- [x] UI Gallery

## Tokens

- [x] Sin colores hardcodeados nuevos
- [x] Tipografía desde theme
- [x] `npm run verify:ui`
