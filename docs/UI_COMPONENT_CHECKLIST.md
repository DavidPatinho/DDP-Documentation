# DDP UI 1.0 — Component Library checklist (Fase 2)

Usar junto con `npm run verify:ui` (`frontend/scripts/verify-design-system.mjs`).

## Componentes

- [x] BaseCard
- [x] HeroCard
- [x] MetricCard
- [x] FocusCard
- [x] InsightCard
- [x] RecommendationCard
- [x] ChartCard
- [x] TechnicalCard
- [x] EmptyCard
- [x] StatusBadge
- [x] SectionHeader
- [x] Metric
- [x] DeltaText
- [x] FocusCallout
- [x] Alert
- [x] MetaGrid
- [x] ListRow
- [x] RankedList
- [x] DataTable
- [x] LoadingBlock
- [x] PageScaffold

## Design System

- [x] Sin colores hardcodeados (`#hex`, `rgb()`, `bg-emerald-500`, etc.) en `components/product`
- [x] Tokens semánticos (`primary`, `success`, `warning`, `danger`, `info`, `delta-*`)
- [x] Tipografía desde `theme/typography.ts`
- [x] Radios / sombras desde theme
- [x] Spacing / layout classes desde theme
- [x] Dark mode vía CSS variables

## Anatomía Metric (UI-D4)

- [x] Label
- [x] Valor principal
- [x] Unidad
- [x] Comparativa (opcional)
- [x] Estado / tendencia (opcional)

## FocusCallout

- [x] Problema principal (`title`)
- [x] Dónde pierde / qué entrenar (`description`, `location`)
- [x] Delta opcional
- [x] Icono
- [x] CTA opcional
- [x] Acento primary (barra + ring)

## Responsive / a11y

- [x] Grids `sm` / `lg` / `xl` en gallery y cards
- [x] `aria-label` / `aria-labelledby` en cards clave
- [x] `role="alert"` en Alert danger/warning
- [x] Delta no solo por color (aria-label con ganancia/pérdida)

## UI Gallery

- [x] Ruta `/ui-gallery`
- [x] Enlace sidebar solo en `import.meta.env.DEV`
- [x] Muestra todos los componentes de producto

## Fuera de alcance Fase 2

- [ ] Integración Dashboard / Sesiones / Briefing / Objetivos / Telemetría → Fase 3+
