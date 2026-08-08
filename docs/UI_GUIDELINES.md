# DDP UI Guidelines 1.0

> Guía visual y de componentes para la etapa **DDP UI 1.0**.
> Alcance exclusivo: frontend (React + Tailwind + shadcn/ui).
> No modifica backend, API, SQLite, Core, motores ni persistencia.

Ante conflicto de producto con este documento, prevalece `DDP_RULES.md`.
Ante conflicto de arquitectura de capas, prevalece `ARCHITECTURE.md`.

**Fuente de tokens (Fase 1):** `frontend/src/theme/` + `frontend/src/index.css`.  
**Shell / scroll (Fase 3):** `AppLayout` — sidebar + header fijos; solo `main-scroll` hace scroll.  
**Ritmo (Fase 3):** zonas 48px · cards 24px · componentes 16px · label→valor 8px.

---

## Decisiones aprobadas

### UI-D1 — Componentes de producto

No diseñamos “botones y divs”. Diseñamos componentes de producto:

`SessionHeroCard` · `MetricCard` · `FocusCard` · `InsightCard` ·
`RecommendationCard` · `TechnicalCard` · `StatusBadge` · `SectionHeader` ·
`PageScaffold`

El piloto debe percibir una herramienta de rendimiento, no un CRUD.

### UI-D2 — Card como unidad básica

La **Card** es la unidad de composición de toda la interfaz.
Las pantallas futuras se construyen reutilizando la biblioteca de cards,
no con listas/grids dispersos ad hoc.

### UI-D3 — Familia Card especializada

```
BaseCard
├── HeroCard
├── MetricCard
├── FocusCard
├── InsightCard
├── RecommendationCard
├── ChartCard
├── TechnicalCard
└── EmptyCard
```

No hay una Card genérica de producto para pantallas: cada variante tiene un
propósito. Implementación: `frontend/src/components/product/`.

### UI-D4 — Anatomía única de métricas

Label → Valor → Unidad → Comparativa? → Estado/tendencia?

Componente canónico: `Metric` (también usado por `MetricCard` / `HeroCard`).

### UI-D5 — Color funcional

| Color | Significado |
|-------|-------------|
| Verde | mejora, éxito, CTA |
| Rojo | problema / pérdida |
| Amarillo | advertencia |
| Azul | información |
| Gris | neutro |

### Una pregunta por pantalla

| Pantalla | Pregunta |
|----------|----------|
| Dashboard | ¿Qué necesito saber hoy? |
| Sesión | ¿Qué ocurrió? |
| Análisis | ¿Dónde pierdo? |
| Driver Briefing | ¿Qué debo entrenar ahora? |
| Objetivos | ¿Estoy mejorando? |
| Trayectoria (Driver Journey) | ¿Cómo he llegado hasta aquí? |

Si una pantalla responde varias preguntas distintas, se reorganiza.

### Jerarquía visual (bloqueada)

1. Hero → 2. KPIs → 3. Diagnóstico → 4. Acciones → 5. Detalles técnicos

Nunca al revés.

---

## 1. Filosofía visual

DDP es una herramienta profesional para pilotos. La interfaz debe facilitar
decisiones en segundos, no administrar registros.

Inspiración de lenguaje (no copia): Porsche Track Precision, Garmin Catalyst,
VRS, MoTeC i2, dashboards modernos de competición.

Principios:

| Principio | Significado |
|-----------|-------------|
| Decisión primero | Jerarquía Hero → KPIs → Diagnóstico → Acciones → Técnico |
| Una pregunta | Cada pantalla responde una sola pregunta del piloto |
| Cards de producto | Composición solo con componentes de la biblioteca |
| Densidad útil | Información densa sin ruido CRUD |
| Premium oscuro | Tema oscuro elegante, deportivo, profesional |
| Métricas protagonistas | Tiempos y deltas mono + tabular-nums |
| Reutilizar, no reinventar | Tokens en `theme/`; cero estilos ad hoc |

---

## 2. Identidad gráfica

| Elemento | Definición |
|----------|------------|
| Nombre | Driver Development Program |
| Corto | DDP |
| Personalidad | Preciso, técnico, calmado, premium |
| Referencia | Dashboards modernos de telemetría / rendimiento (inspiración, no copia) |
| Marca en UI | Monograma `DDP` + nombre corto en sidebar; evitar logos complejos |
| Acento | Verde (rendimiento / foco positivo) |
| Alerta | Rojo/ámbar solo para error, warning e incidencia |
| Superficie | Paneles y cards elevados sobre fondo oscuro |

La identidad no usa púrpura, ni glow neon, ni pill clusters decorativos.

---

## 3. Paleta de colores

Todas las pantallas consumen tokens CSS. Prohibido hardcodear colores
arbitrarios (`bg-emerald-500`, `text-[11px]` con color suelto, hex sueltos)
fuera de `index.css` o tokens de gráfico.

### 3.1 Tokens semánticos (objetivo UI 1.0)

| Token | Rol | Notas |
|-------|-----|-------|
| `--background` | Lienzo de app | Oscuro profundo |
| `--foreground` | Texto principal | Alto contraste, casi blanco |
| `--card` | Superficie de panel/card | Un escalón más claro que el fondo |
| `--card-foreground` | Texto sobre card | Igual o cercano a foreground |
| `--popover` | Menús / overlays | Igual o ligeramente sobre card |
| `--primary` | Acción principal / marca | **Verde** (no azul actual) |
| `--primary-foreground` | Texto sobre primary | Oscuro o blanco según contraste WCAG |
| `--secondary` | Superficie secundaria | Neutro oscuro |
| `--muted` | Fondos suaves / chips | Neutro |
| `--muted-foreground` | Labels, metadatos | Gris medio |
| `--accent` | Hover / selección sutil | Neutro con leve tinte |
| `--destructive` | Error / peligro | Rojo |
| `--border` | Bordes | Bajo contraste (~8–12% blanco) |
| `--input` | Borde/fondo de inputs | Similar a border |
| `--ring` | Focus ring | Alineado a primary |
| `--sidebar*` | Navegación | Más oscuro que background |

### 3.2 Tokens de dominio (Fase 1 — implementados)

| Token | Uso |
|-------|-----|
| `--success` / `--success-foreground` | Estado OK, mejoras |
| `--warning` / `--warning-foreground` | Aviso, telemetría parcial |
| `--danger` / `--danger-foreground` | Error, incidencia |
| `--info` / `--info-foreground` | Información neutra |
| `--delta-gain` | Más rápido vs referencia |
| `--delta-loss` | Pérdida de tiempo vs referencia |
| `--chart-1` … `--chart-5` | Series Recharts |
| `--chart-grid` / `--chart-axis` | Rejilla y ejes |

Definiciones OKLCH: `frontend/src/theme/colors.ts` y `frontend/src/theme/charts.ts`.

### 3.3 Primary

**Verde de rendimiento** (`oklch` hue ~148). Sustituye el azul provisional de v0.x.

Convención de deltas (bloqueada): pérdida → `delta-loss`; ganancia → `delta-gain`.

---

## 4. Tipografía

### 4.1 Familias (Fase 1 — implementadas)

| Rol | Familia | Uso |
|-----|---------|-----|
| UI sans | **Plus Jakarta Sans** (variable) | Títulos, body, navegación |
| Mono / métricas | **JetBrains Mono** | Tiempos, deltas, hashes, IDs |

Carga: `@fontsource` en `main.tsx`. Escala: `frontend/src/theme/typography.ts`.

### 4.2 Escala

| Token | Tamaño | Uso |
|-------|--------|-----|
| `hero` | 30 px mono | Métrica hero |
| `kpi` | 24 px mono | MetricCard |
| `kpi-sm` | 18 px mono | Deltas en listas |
| `h1` | 22 px | Título de página |
| `h2` | 15 px | Título de card / sección |
| `h3` | 13 px | Subsección |
| `body` | 14 px | Texto general |
| `body-sm` | 13 px | Texto secundario |
| `caption` | 12 px | Metadatos |
| `label` | 11 px uppercase tracking | Etiquetas de métrica |
| `mono` | 13 px | IDs / hashes técnicos |

Reglas:

- Tiempos y deltas siempre `font-mono tabular-nums`.
- Labels de métrica: `text-xs uppercase tracking-wide text-muted-foreground`.
- No inventar tamaños one-off (`text-[11px]`) fuera de la escala.

---

## 5. Espaciados

Base **4 px**. Preferir escala Tailwind:

| Token | Valor | Uso |
|-------|-------|-----|
| `1` | 4 px | Micro gaps internos |
| `2` | 8 px | Icono–texto, chips |
| `3` | 12 px | Controles compactos |
| `4` | 16 px | Padding interno de card densa |
| `5` | 20 px | Gaps medios |
| `6` | 24 px | Padding de card estándar / page section |
| `8` | 32 px | Separación entre secciones mayores |
| `10` | 40 px | Respiro de layout |
| `12` | 48 px | Empty states |

Convenciones de página:

- Contenedor principal: `p-6` (desktop); `p-4` en viewport estrecho.
- Stack de página: `space-y-6` (estándar) o `space-y-8` solo entre bloques de decisión mayores.
- Interior de card: `p-5` o `p-6`.
- Header de card: `px-5 py-4` + borde inferior.

---

## 6. Radios

| Token | Valor | Uso |
|-------|-------|-----|
| `--radius` | `0.75rem` (12 px) | Base del sistema |
| `radius-sm` | base − 4 | Chips, inputs densos |
| `radius-md` | base − 2 | Controles |
| `radius-lg` | base | Cards, paneles |
| `radius-xl` | base + 4 | Modales / sheets |

Evitar `rounded-full` salvo indicadores de estado (dot) e iconos circulares
mínimos. No pills decorativos.

---

## 7. Sombras

Tema oscuro: sombra suave + borde, no multi-layer glow.

| Nivel | Receta orientativa | Uso |
|-------|--------------------|-----|
| none | Solo borde | Listas internas, filas |
| sm | `0 1px 2px rgb(0 0 0 / 25%)` | Controles elevados |
| md | `0 4px 12px rgb(0 0 0 / 35%)` | Cards principales |
| lg | `0 8px 24px rgb(0 0 0 / 45%)` | Popovers, diálogos |

En dark mode el **borde** aporta más jerarquía que la sombra. No apilar
sombras grandes + glow primary.

---

## 8. Elevaciones

| Nivel | Superficie | Ejemplo |
|-------|------------|---------|
| 0 | `--background` | Canvas |
| 1 | `--sidebar` / rail | Navegación |
| 2 | `--card` | Paneles y cards |
| 3 | `--popover` | Menús, tooltips, sheets |
| 4 | Modal / dialog | Confirmaciones |

Una card nunca debe “flotar” sobre otra sin cambio de elevación o borde claro.

---

## 9. Grid

| Contexto | Grid |
|----------|------|
| App shell | Sidebar fija + `SidebarInset` fluido |
| Contenido | Max width opcional `max-w-6xl` / `max-w-7xl` en páginas densas; dashboard puede ser full-bleed |
| KPIs | `grid gap-4 sm:grid-cols-2 lg:grid-cols-3` (o 4 en dashboard) |
| Split decisión | `grid gap-6 lg:grid-cols-12` con columnas 7/5 o 8/4 |
| Listas | Una columna; tablas para datos tabulares reales |

Breakpoints de referencia (Tailwind):

| Prefijo | Ancho |
|---------|-------|
| default | < 640 |
| `sm` | ≥ 640 |
| `md` | ≥ 768 (sidebar desktop / mobile sheet) |
| `lg` | ≥ 1024 |
| `xl` | ≥ 1280 |

---

## 10. Botones

Usar `@/components/ui/button`. No botones HTML sueltos con clases ad hoc.

| Variant | Uso |
|---------|-----|
| `default` (primary) | Acción principal de la vista (Analizar, Importar, Interpretar) |
| `outline` | Secundaria (Volver, Vaciar cola) |
| `secondary` | Alternativa menos enfatizada |
| `ghost` | Iconos / acciones terciarias en filas |
| `destructive` | Destructivo explícito |
| `link` | Navegación textual inline |

| Size | Uso |
|------|-----|
| `default` | Estándar |
| `sm` | Filas densas / tablas |
| `lg` | CTA hero (dashboard / empty con acción) |
| `icon*` | Solo icono con `aria-label` |

Reglas:

- Una sola acción primary visible por región.
- Loading: icono `LoaderCircle` + label (“Calculando”, “Importando”); botón `disabled`.
- No usar primary azul residual tras Fase 1.

---

## 11. Cards

Las cards son el contenedor principal de decisión (excepción consciente al
patrón “sin cards” de landing pages: aquí son paneles de producto).

Anatomía:

```
┌─ Card ─────────────────────────────────────────┐
│ Header: título + descripción opcional + actions │
│────────────────────────────────────────────────│
│ Body: métricas / listas / charts                │
│ Footer opcional: CTA o meta                     │
└────────────────────────────────────────────────┘
```

Estilo:

- `rounded-lg border border-border bg-card shadow-sm` (o elevación md).
- Header con `border-b` y tipografía `h2`.
- No cards anidadas sin necesidad.
- No card en el shell (sidebar/header); sí en el contenido.

Variantes:

| Variante | Uso |
|----------|-----|
| `Card` | Estándar |
| `CardMetric` | KPI destacado |
| `CardFocus` | Próximo foco / problema principal (énfasis primary sutil) |
| `CardMuted` | Información secundaria / técnica |

`ChartPanel` existente se alinea a esta anatomía o se absorbe en `Card`.

---

## 12. Inputs

Usar `@/components/ui/input` (+ futuros `Label`, `Select`, `Textarea`, `Checkbox`).

Reglas:

- Altura alineada a botones (`h-8` / `h-9` según tamaño del sistema).
- Siempre label visible o `aria-label`.
- Placeholder como ejemplo, no como label.
- Errores: `aria-invalid` + mensaje `text-destructive` / `FieldError`.
- Grupos: label arriba, help text abajo, gap `2`.

Formularios de configuración futuros: una columna, anchos máximos, no grids
de inputs densos estilo CRUD.

---

## 13. Tablas

Hoy las “tablas” son listas `ul` con `divide-y`. En UI 1.0:

| Caso | Componente |
|------|------------|
| Top 3 pérdidas / insights | `ListRow` / `RankedList` (no tabla) |
| Cola de importación | `DataList` o tabla simple |
| Vueltas / hallazgos completos | `Table` (shadcn) dentro de card o collapsible |
| Trazabilidad técnica | Definition list (`MetaGrid`) o tabla compacta |

Reglas de tabla:

- Header sticky opcional en listas largas.
- Celdas numéricas a la derecha, mono tabular.
- Filas con hover `bg-muted/40`.
- Densidad `sm` en paneles técnicos; `md` en vistas piloto.
- No zebra agresiva; borde sutil basta.

---

## 14. Métricas

Componente clave del producto: `Metric` / `MetricCard`.

Anatomía:

```
LABEL (uppercase, muted)
VALUE (mono, grande, tabular)
HINT  (caption opcional)
```

Tipos:

| Tipo | Ejemplo |
|------|---------|
| Tiempo | `1:32.447` |
| Delta | `+0.320` / `−0.120` (color loss/gain) |
| Texto decisión | “Frenada T3” |
| Contador | `12` vueltas válidas |
| Estado | badge + label |

Reglas:

- Delta positivo de **pérdida** usa color danger; ganancia usa success
  (definir convención una sola vez en Fase 1 y documentarla en código).
- Máximo 3–4 métricas hero por viewport inicial.
- En sesión: Ideal, Consistencia, Curva prioritaria / Foco siguiente son
  candidatos a hero.

---

## 15. Gráficos

Stack: **Recharts** (ya en dependencias) + contenedor `ChartPanel` / `Card`.

Reglas:

- Colores solo desde `--chart-*`.
- Fondo transparente; grid con `--track-grid`.
- Tooltip estilo popover (fondo card, borde, texto sm).
- Ejes: muted, sin ruido; tipografía caption.
- Altura por defecto 240–320 px; no charts “postage stamp”.
- Empty: empty state dentro del panel, no chart vacío engañoso.
- Animación de entrada breve; sin animaciones continuas distractoras.

---

## 16. Estados

Estados de dominio unificados (análisis / interpretación / importación):

| Estado UI | Presentación |
|-----------|--------------|
| `idle` / `none` | CTA + copy corto |
| `loading` | Skeleton del bloque (no solo texto) |
| `running` | Botón busy + progress sutil |
| `complete` | Contenido + badge “Listo” |
| `unavailable` | Warning + explicación accionable |
| `error` | Alert destructive + reintento si aplica |
| `empty` | EmptyState con CTA |

Componente propuesto: `StatusBadge` (mapa de estado → color/label).
Eliminar labels duplicados `statusLabel()` locales a favor de un mapa único.

---

## 17. Empty states

Usar siempre `EmptyState` (extender el actual):

Props objetivo:

- `icon`
- `title`
- `description`
- `action?` (CTA)
- `size?: "sm" | "md" | "lg"`

Reglas:

- Dashed border + fondo `bg-card/40` (o card sólida muted).
- Un mensaje, una acción.
- Telemetría “Cola vacía” debe migrar a `EmptyState` (hoy es markup inline).
- Páginas placeholder (Dashboard, Piloto, Informes, Config) mantienen EmptyState
  hasta su fase, pero con copy orientado a decisión y CTA cuando exista destino.

---

## 18. Skeleton loading

Usar `@/components/ui/skeleton` (hoy casi sin uso en páginas).

Patrones:

| Vista | Skeleton |
|-------|----------|
| Sesión | Header metrics (3) + 2 cards |
| Dashboard | Fila de 4 KPI + 2 paneles |
| Lista | 4–6 filas |
| Texto briefing | 3 líneas de distinto ancho |

Reglas:

- Nunca dejar solo “Cargando…” como UI final de Fase 3+.
- Skeleton con la misma geometría que el contenido real.
- Evitar spinners de página completa salvo acciones cortas en botón.

---

## 19. Animaciones

Principio: presencia y jerarquía, no ruido.

Permitido:

- Fade/slide corto en entrada de cards (150–250 ms).
- `animate-spin` en loaders de acción.
- `animate-pulse` en skeletons.
- Rotación del chevron en collapsibles.
- Hover transitions en botones/filas (`transition-colors`).

Evitar:

- Parallax, glow pulses, confetti, animaciones infinitas decorativas.
- Movimiento en datos críticos que dificulte lectura de métricas.

Motion tokens sugeridos:

| Token | Valor |
|-------|-------|
| `duration-fast` | 120 ms |
| `duration-normal` | 200 ms |
| `easing-standard` | `ease-out` |

---

## 20. Responsive

Producto principal: **desktop Tauri (Windows)**. Mobile/tablet es secundario
pero el shell ya soporta sidebar sheet (`useIsMobile`, breakpoint 768).

| Ancho | Comportamiento |
|-------|----------------|
| < 768 | Sidebar offcanvas; stacks verticales; KPIs 1 col |
| 768–1023 | Sidebar icon/expand; KPIs 2 col |
| ≥ 1024 | Sidebar completa; KPIs 3–4 col; splits 12-col |

Reglas:

- Touch targets mínimos 40 px en controles críticos.
- No ocultar el “próximo foco” en mobile; sí se puede colapsar lo técnico.
- Tablas anchas: scroll horizontal dentro de la card, no romper el shell.

---

## 21. Accesibilidad

| Requisito | Práctica |
|-----------|----------|
| Contraste | Texto y métricas AA sobre dark |
| Foco | Ring visible (`--ring`); no `outline-none` sin reemplazo |
| Semántica | `h1` único por página; secciones con `aria-labelledby` |
| Iconos | `aria-label` en botones icon-only |
| Errores | `role="alert"` (ya usado; mantener) |
| Collapsibles | Preferir patrón accesible; si `<details>`, summary claro |
| Estado | No transmitir solo por color (delta + signo + label) |
| Motion | Respetar `prefers-reduced-motion` en animaciones de entrada |

---

## 22. Iconografía

- Librería: **lucide-react** (ya en uso).
- Estilo: stroke consistente, tamaño default 16 (`size-4`); headers 20 (`size-5`).
- Nav: un icono por ítem (`theme/navigation.ts`).
- No mezclar emojis en UI de producto.
- Collapsibles: chevron Lucide, no carácter `▾`.
- Status dots: círculo 8 px con token success/warning/danger (no `bg-emerald-500` suelto).

---

## 23. Componentes reutilizables

### 23.1 Primitivos UI (shadcn / base-ui) — existentes

| Componente | Ruta | Estado |
|------------|------|--------|
| Button | `components/ui/button.tsx` | OK |
| Input | `components/ui/input.tsx` | OK |
| Separator | `components/ui/separator.tsx` | OK |
| Skeleton | `components/ui/skeleton.tsx` | Infra OK, uso pobre |
| Tooltip | `components/ui/tooltip.tsx` | OK |
| Sidebar / Sheet | `components/ui/sidebar.tsx`, `sheet.tsx` | OK |
| ScrollArea | `components/ui/scroll-area.tsx` | OK |

### 23.2 Layout / app — existentes

| Componente | Ruta | Notas |
|------------|------|-------|
| AppLayout | `layouts/AppLayout.tsx` | Shell |
| AppSidebar | `components/AppSidebar.tsx` | Nav |
| AppHeader | `components/AppHeader.tsx` | Título + status |
| PageHeader | `components/PageHeader.tsx` | Título de página |
| EmptyState | `components/EmptyState.tsx` | Ampliar con CTA |
| ChartPanel | `components/ChartPanel.tsx` | Sin uso; alinear a Card |
| CollapsibleSection | `components/CollapsibleSection.tsx` | Técnico |

### 23.3 Dominio sesión — existentes (refactor visual en fases)

| Componente | Ruta |
|------------|------|
| SessionAnalysisPanel | `components/SessionAnalysisPanel.tsx` |
| SessionInterpretationPanel | `components/SessionInterpretationPanel.tsx` |
| SessionTechnicalPanel | `components/SessionTechnicalPanel.tsx` |

### 23.4 Component Library de producto (Fase 2 — UI-D1)

| Componente | Responsabilidad |
|------------|-----------------|
| `PageScaffold` | Contenedor de página: pregunta, hierarchy stack, max-width |
| `SectionHeader` | Título de zona + actions |
| `SessionHeroCard` | Hero de sesión (qué ocurrió) |
| `MetricCard` | KPI label / value / hint |
| `FocusCard` | Próximo foco / problema principal |
| `InsightCard` | Insight accionable |
| `RecommendationCard` | Recomendación + criterio de éxito |
| `TechnicalCard` | Detalle técnico / collapsible |
| `StatusBadge` | Estados de proceso (análisis, import, etc.) |

Primitivos de soporte (si hacen falta bajo las cards): `Alert`, `DeltaText`,
`ListRow`, `DataTable`, skeletons.

### 23.5 Reglas de reutilización

1. Si un patrón aparece ≥ 2 veces, se extrae.
2. Los paneles de dominio componen primitivos; no reimplementan bordes/spacing.
3. Tokens solo en CSS theme; componentes solo clases semánticas.
4. Sin lógica de dominio nueva en la capa visual (solo presentación).

---

## Apéndice A — Inventario de pantallas (estado actual)

| Ruta | Página | Madurez visual | Contenido |
|------|--------|----------------|-----------|
| `/` | Dashboard | Placeholder | EmptyState |
| `/sessions` | Sesiones (lista) | Placeholder | EmptyState + CTA a Telemetría |
| `/sessions/:id` | Detalle de sesión | Funcional, estilo “doc/CRUD” | Resumen + Análisis + Interpretación + Técnico |
| `/telemetry` | Telemetría / import | Funcional, lista plana | Cola .ibt + acciones |
| `/driver` | Piloto | Placeholder | EmptyState |
| `/reports` | Informes | Placeholder | EmptyState |
| `/settings` | Configuración | Placeholder | EmptyState |

## Apéndice B — Fuera de alcance UI 1.0

- Cambios de API, contratos JSON, motores, SQLite, versiones de motores.
- Nueva lógica de dominio en frontend.
- Rediseño de flujos de negocio (solo presentación y composición visual).

## Apéndice C — Criterio de “fase terminada”

Una fase está cerrada cuando:

1. Sus entregables existen en código o docs.
2. No quedan TODOs bloqueantes de esa fase.
3. Se ha revisado visualmente en dark theme desktop.
4. Se obtiene aprobación explícita antes de la siguiente fase.
