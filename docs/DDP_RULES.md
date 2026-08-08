# DDP — Reglas del Proyecto

> Documento maestro de arquitectura, diseño y comportamiento.
> Toda decisión técnica o de producto debe alinearse con estas reglas.
> Si una propuesta las contradice, debe rechazarse o replantearse.

---

## Filosofía del proyecto

DDP **no es una aplicación web**.

DDP es un **software profesional de escritorio** para el entrenamiento y desarrollo de pilotos de simracing.

Debe sentirse como herramientas de referencia en su categoría:

- **MoTeC** — precisión técnica, datos como protagonistas
- **SimHub** — utilidad inmediata, interfaz funcional
- **TradingView** — visualización clara de series temporales
- **Adobe** — consistencia visual, jerarquía, sensación de producto maduro

### Lo que DDP debe ser

- Una herramienta de ingeniería de pista
- Una aplicación nativa de escritorio (Tauri)
- Un entorno de trabajo prolongado, no una visita rápida
- Una interfaz densa en utilidad, no en decoración

### Lo que DDP nunca debe ser

- Una landing page
- Un dashboard genérico de SaaS
- Una web responsive adaptada a ventana
- Un chatbot disfrazado de asistente
- Un panel saturado de métricas sin contexto

**Regla absoluta:** si una pantalla puede confundirse con una página web, está mal diseñada.

---

## Objetivo

Ayudar a un piloto a **mejorar utilizando datos**.

El sistema existe para convertir telemetría, sesiones y contexto en **decisiones accionables**, no en ruido informativo.

### Prioridades del producto

1. **Claridad** — el piloto entiende qué pasó y por qué
2. **Progreso** — el piloto ve cómo evoluciona
3. **Acción** — el piloto sabe qué trabajar en la siguiente sesión

### Lo que el sistema no debe hacer

- Mostrar todos los datos disponibles "por si acaso"
- Competir por atención con gráficos decorativos
- Obligar al usuario a interpretar raw data sin guía
- Generar informes extensos que nadie lee

**Principio rector:** menos información visible, más valor por dato mostrado.

La IA debe **simplificar**, no amplificar el ruido.

---

## Principios

### Diseño

| Principio | Descripción |
|-----------|-------------|
| Minimalismo | Solo lo necesario. Cada elemento compite por atención. |
| Espacio | Márgenes generosos. Respiración visual. Nada apretado. |
| Tema oscuro | Único tema de producción. Reduce fatiga en sesiones largas. |
| Animaciones suaves | Transiciones sutiles. Nunca distractoras. Funcionales. |

### Código

| Principio | Descripción |
|-----------|-------------|
| Componentes reutilizables | Si se usa más de una vez, es componente. |
| Cero duplicación | DRY estricto. Extraer, no copiar. |
| Arquitectura limpia | Capas definidas. Responsabilidades claras. |
| Mantenibilidad | El código debe poder evolucionar durante años. |

### Tamaño y complejidad

- No crear componentes enormes (>200 líneas es señal de alerta)
- No crear dependencias innecesarias
- Siempre priorizar **claridad** sobre cleverness
- Preferir composición sobre herencia

---

## Stack tecnológico

Stack **obligatorio**. No sustituir tecnologías sin aprobación explícita.

| Capa | Tecnología | Rol |
|------|------------|-----|
| Frontend UI | React | Interfaz de usuario |
| Tipado | TypeScript | Seguridad y contratos |
| Desktop | Tauri | Contenedor nativo Windows |
| Backend API | Python FastAPI | Lógica de servidor, procesamiento |
| Base de datos | SQLite | Persistencia local |
| Estilos | Tailwind CSS | Utility-first, tokens de diseño |
| Componentes | shadcn/ui | Base UI consistente y customizable |
| Gráficos | Recharts | Visualización de series temporales |
| Datos | Pandas | Manipulación de telemetría y sesiones |
| Cálculo | NumPy | Operaciones numéricas |

### Responsabilidades por capa

```
┌─────────────────────────────────────────┐
│  Tauri (shell nativo)                   │
├─────────────────────────────────────────┤
│  React (UI + navegación + estado view)  │
├─────────────────────────────────────────┤
│  FastAPI (API + lógica de negocio)      │
├─────────────────────────────────────────┤
│  Pandas / NumPy (procesamiento datos)   │
├─────────────────────────────────────────┤
│  SQLite (persistencia)                  │
└─────────────────────────────────────────┘
```

- **React** renderiza y gestiona interacción. No procesa telemetría.
- **FastAPI** expone endpoints y orquesta lógica. No conoce el DOM.
- **Pandas/NumPy** viven en el backend. Nunca en el frontend.
- **SQLite** es la única base de datos. Local, embebida, sin servidor externo.

---

## Convenciones

### El modelo de dominio es la referencia oficial

`docs/DATA_MODEL.md` está **congelado en la versión 1.0** y es la referencia
oficial del dominio para todo el proyecto.

1. **Todo se deriva del modelo, nunca al contrario.** El esquema de SQLite, los
   tipos de TypeScript, los modelos de FastAPI y las estructuras del Core se
   derivan de `DATA_MODEL.md`. Si el código y el documento discrepan, el
   documento tiene razón.
2. **Los nombres del modelo son los nombres del código.** Una entidad se llama
   igual en el documento, en la base de datos, en el backend y en los tipos del
   frontend. Sin sinónimos ni abreviaturas locales.
3. **Un dato que puede calcularse no se almacena como hecho.** Las entidades
   derivadas son borrables y recalculables; las fundamentales son la fuente de
   verdad.
4. **Un dato específico de una plataforma** entra por `external_ids[]`,
   `attributes{}` o `raw_payload`, nunca como atributo nuevo de una entidad.
5. **Modificar entidades, relaciones o clasificación abre una versión 1.1** del
   documento, con su propia auditoría. No se modifica el modelo para acomodar una
   dificultad de implementación.

El vocabulario del dominio (sección 14 del modelo) es el que debe aparecer en la
interfaz y en los informes al piloto.

### Cadena de derivación

```
DATA_MODEL.md v1.0  ──>  DATABASE_DESIGN.md  ──>  SQLite
   (qué existe)          (cómo se guarda)        (implementación)
```

`docs/DATABASE_DESIGN.md` es la referencia oficial para implementar SQLite, y está
congelado en la versión 1.0. La cadena nunca se recorre en sentido contrario: una
dificultad de implementación no justifica cambiar el diseño, y una dificultad de
diseño no justifica cambiar el modelo.

**Durante la implementación no se hace ninguna modificación estructural.** Ni una
tabla, ni una columna, ni una clave ajena que no esté en el diseño. Si aparece una
necesidad estructural, el desarrollo se detiene y se abre una versión 1.1 del
documento correspondiente. Modificar el esquema mientras se programa es lo que
convierte un diseño en una acumulación de parches.

### Estructura de carpetas

```
DDP/
├── frontend/          # React + Tauri
│   └── src/
│       ├── components/    # Componentes reutilizables
│       ├── layouts/       # Estructuras de página
│       ├── pages/         # Páginas independientes (una por ruta)
│       ├── services/      # Comunicación con API
│       ├── hooks/         # Lógica reutilizable de UI
│       ├── types/         # Tipos TypeScript compartidos
│       ├── theme/         # Tokens, navegación, constantes visuales
│       └── assets/        # Recursos estáticos del frontend
├── backend/           # FastAPI
├── database/          # Archivos SQLite y migraciones
├── docs/              # Documentación del proyecto
└── assets/            # Recursos compartidos del proyecto
```

### Reglas de organización

1. **Todos los componentes reutilizables** viven en `components/`
2. **Todas las páginas son independientes** — una carpeta/archivo por ruta, sin lógica cruzada
3. **Separación estricta UI / lógica:**
   - UI → `components/`, `layouts/`, `pages/`
   - Lógica de datos → `services/`, `hooks/`
   - Tipos → `types/`
4. **Toda la lógica de telemetría debe estar aislada** en el backend:
   - Frontend: visualización y selección
   - Backend: parsing, cálculo, agregación, comparación
5. **shadcn/ui** en `components/ui/` — no modificar directamente; extender en `components/`

### Nomenclatura

| Elemento | Convención | Ejemplo |
|----------|------------|---------|
| Componentes | PascalCase | `SessionCard.tsx` |
| Hooks | camelCase con `use` | `useSessionData.ts` |
| Servicios | camelCase | `sessionService.ts` |
| Tipos | PascalCase | `Session`, `TelemetryChannel` |
| Páginas | PascalCase + Page | `DashboardPage.tsx` |
| Constantes | UPPER_SNAKE | `MAX_LAP_COUNT` |

### Imports

- Usar alias `@/` para rutas internas
- Orden: externos → internos → tipos → estilos
- Nunca importar lógica de backend en frontend directamente

---

## Paleta

Paleta **obligatoria**. No introducir colores fuera de esta gama sin justificación.

| Token | Uso | Valor orientativo |
|-------|-----|-------------------|
| **Negro** | Fondo principal, sidebar | `#0a0a0b` – `#111113` |
| **Gris oscuro** | Superficies, cards, bordes | `#1a1a1e` – `#2a2a2e` |
| **Azul eléctrico** | Acciones primarias, selección, foco | `#3b82f6` – `#2563eb` |
| **Verde** | Mejoras, deltas positivos, progreso | `#22c55e` – `#16a34a` |
| **Rojo** | Errores **únicamente** | `#ef4444` – `#dc2626` |

### Reglas de color

- El rojo **nunca** se usa para decoración, alertas leves o "llamar la atención"
- El verde **solo** indica mejora o valor positivo medible
- El azul eléctrico es el único color de acento interactivo
- Texto secundario: gris medio sobre gris oscuro, nunca blanco puro en párrafos largos
- Gráficos: máximo 3 colores simultáneos por vista

---

## Tipografía

| Propiedad | Valor |
|-----------|-------|
| Familia principal | **Inter** |
| Familia mono | JetBrains Mono o Cascadia Code (datos numéricos) |

### Jerarquía visual

| Nivel | Tamaño | Peso | Uso |
|-------|--------|------|-----|
| H1 | 20–24px | 600 | Título de página |
| H2 | 16–18px | 600 | Sección |
| H3 | 14px | 500 | Subsección, label de panel |
| Body | 13–14px | 400 | Texto general |
| Caption | 11–12px | 400 | Metadatos, timestamps |
| Data | 13–14px mono | 500 | Valores numéricos, tiempos |

### Reglas tipográficas

- Jerarquía clara: el usuario identifica la estructura sin leer
- Datos numéricos siempre en fuente mono
- No más de 3 niveles de heading visibles simultáneamente
- Line-height generoso (1.5 mínimo en body)

---

## Filosofía de UX

### Principios

1. **Mostrar primero lo importante** — above the fold = lo que el piloto necesita ahora
2. **Ocultar la complejidad** — detalle bajo demanda (expand, drill-down, tabs)
3. **El usuario nunca debe sentirse perdido** — siempre visible: dónde estoy, qué estoy viendo, cómo volver

### Patrones obligatorios

| Patrón | Aplicación |
|--------|------------|
| Empty state | Toda vista sin datos muestra contexto, no pantalla en blanco |
| Loading state | Skeleton, nunca spinner genérico a pantalla completa |
| Error state | Mensaje claro + acción de recuperación |
| Navegación persistente | Sidebar siempre accesible |
| Breadcrumb contextual | En vistas anidadas (sesión → vuelta → sector) |

### Anti-patrones prohibidos

- Modales encadenados
- Scroll infinito sin ancla
- Tooltips como única fuente de información crítica
- Formularios de más de 5 campos visibles a la vez
- Notificaciones toast para información importante

### Flujo de atención

```
Contexto → Insight → Acción
   ↓          ↓         ↓
"Qué sesión" "Qué pasó" "Qué hacer"
```

---

## Filosofía IA

La IA en DDP debe comportarse como un **ingeniero de pista senior**, no como un chatbot.

### Lo que la IA debe hacer

| Comportamiento | Descripción |
|----------------|-------------|
| Analizar | Procesar datos y detectar patrones relevantes |
| Explicar | Traducir datos técnicos a lenguaje comprensible |
| Enseñar | Conectar el dato con la técnica de conducción |
| Priorizar | Señalar 1–3 puntos clave, no un informe completo |
| Contextualizar | Relacionar con sesiones anteriores y objetivos del piloto |

### Lo que la IA nunca debe hacer

- Responder con texto innecesario o genérico
- Actuar como asistente conversacional ("¿En qué puedo ayudarte?")
- Generar párrafos largos sin estructura
- Inventar datos o inferencias sin base en telemetría real
- Usar tono informal, emojis o lenguaje de marketing

### Formato de salida IA

```
[INSIGHT]     Una frase: qué pasó
[EVIDENCIA]   2–3 datos concretos que lo sustentan
[ACCIÓN]      Qué trabajar en la próxima sesión
```

Máximo 150 palabras por insight. Si necesita más, es un informe estructurado, no un chat.

---

## Calidad

### Estándares de código

| Criterio | Umbral |
|----------|--------|
| Tamaño de componente | < 200 líneas |
| Tamaño de función | < 40 líneas |
| Complejidad ciclomática | Baja. Extraer si hay más de 3 branches |
| Cobertura de tipos | TypeScript strict. Sin `any` |
| Dependencias nuevas | Requieren justificación documentada |

### Revisiones obligatorias

Antes de integrar cualquier cambio, verificar:

- [ ] ¿Respeta la paleta y tipografía?
- [ ] ¿Separa UI de lógica?
- [ ] ¿Es reutilizable o duplica código existente?
- [ ] ¿La telemetría vive en backend?
- [ ] ¿Parece app de escritorio o parece web?
- [ ] ¿La IA simplifica o complica?

### Deuda técnica

- No acumular TODOs sin ticket/issue
- No dejar código comentado "por si acaso"
- No crear abstracciones prematuras
- Refactorizar cuando un componente supera 200 líneas

---

## Referencia rápida

```
¿Es web?          → NO
¿Satura?          → NO
¿Duplica?         → NO
¿Procesa en FE?   → NO (telemetría)
¿Parece chatbot?  → NO
¿Claro en 3s?     → SÍ
¿Mantenible?      → SÍ
¿Está en el modelo?  → SÍ (docs/DATA_MODEL.md v1.0)
```

---

*Última actualización: documento fundacional del proyecto DDP.*
