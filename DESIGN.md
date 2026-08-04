# DESIGN.md — Terminal Móvil de Monitorización

**Versión:** 1.0 | **Fecha:** 31 julio 2026 | **Base:** spec v0.5 + modelo IBKR Mobile aprobado (home=watchlist, quote details, chart fullscreen) | **Default:** tema oscuro, bottom bar 3 tabs

> Documento de diseño para alimentar los prompts de build (F3/F4). Define tokens, componentes y pantallas. Todo es mobile-first 360–430px; desktop es escala-up centrado con max-width 480px.

## 1. Principios de diseño

1. **Densidad IBKR:** máximo dato útil por pantalla sin ruido; nada decorativo.
2. **Cero fricción:** abrir app → precios en < 2s; cualquier símbolo a 1 toque.
3. **Oscuro por defecto:** patrón de terminales de trading; claro como toggle en Más.
4. **Estado vivo:** la UI respira con el mercado — flash en ticks, badge de conexión siempre visible.
5. **Una mano:** toda acción primaria alcanzable con el pulgar (bottom bar, CTAs bajos).

## 2. Design tokens (CSS variables)

### Color — tema oscuro (default)

| Token | Valor | Uso |
|---|---|---|
| `--bg-primary` | `#0D1117` | Fondo de app |
| `--bg-surface` | `#161B22` | Cards, filas, bottom bar |
| `--bg-elevated` | `#21262D` | Modales, chips activos |
| `--border-subtle` | `#30363D` | Separadores de fila |
| `--text-primary` | `#E6EDF3` | Precios, tickers |
| `--text-secondary` | `#8B949E` | Nombres de empresa, labels |
| `--accent-up` | `#2EA043` | Ticks al alza, % positivo |
| `--accent-down` | `#F85149` | Ticks a la baja, % negativo |
| `--accent-primary` | `#388BFD` | CTAs, links, tab activo |
| `--accent-warn` | `#D29922` | Badge "Datos retrasados" |

Tema claro: mismos tokens invertidos (bg `#FFFFFF`/`#F6F8FA`, text `#1F2328`/`#57606A`, accents con -10% luminosidad). Cambio sin flicker vía `data-theme` en `<html>`.

### Tipografía

- **Familia:** `Inter` (UI) + `JetBrains Mono` o `Roboto Mono` (precios y números — alineación tabular `font-variant-numeric: tabular-nums` obligatoria para que las columnas no bailen en cada tick).
- **Escala:** ticker 16px/600 · precio 17px/600 mono · % cambio 14px/500 mono · nombre empresa 13px/400 secondary · stats label 12px/500 uppercase secondary · stats valor 15px/500 mono.
- Mínimo 14pt en cuerpo; nunca por debajo de 12px (labels).

### Espaciado y forma

- Base 4px. Filas de watchlist: padding 12px 16px, altura ~64px.
- Radio: 8px cards/chips, 12px CTAs, full en pills.
- Targets táctiles ≥ 44×44px en TODO lo interactivo.
- Bottom bar: 56px + safe-area inset.

### Motion

- Transiciones de pantalla: slide horizontal 250ms `ease-out` (push) / `ease-in` (back).
- Flash de tick: background de la celda de precio a `accent-up/down` al 15% opacidad, fade 400ms.
- Pull-to-refresh: spinner nativo-like, sin animaciones custom.
- Respeta `prefers-reduced-motion`.

## 3. Componentes (catálogo para prompts)

| Componente | Spec |
|---|---|
| `PriceRow` | Fila watchlist: ticker + nombre (col izq), precio + % (col dcha, mono). Flash on tick. Tap → quote. Altura 64px. |
| `ConnectionBadge` | Pill 10px dot + texto: "En tiempo real" (verde) / "Datos retrasados" (ámbar) / "Sin conexión" (rojo). Posición fija sobre la bottom bar. |
| `WatchlistTabs` | Selector horizontal scrollable de listas + contador. Swipe cambia lista activa. |
| `BottomTabBar` | 3 tabs: Watchlist (icono lista) / Buscar (lupa) / Más (⋯). Tab activo en `--accent-primary`. |
| `StatCell` | Label uppercase + valor mono. Grid 2 col en quote details. |
| `TimeframePills` | 5m · 1h · 1D. Activa con bg `--bg-elevated` + borde accent. |
| `IndicatorChip` | Chip togglable: SMA EMA RSI MACD VOL. Activo = borde accent + dot de color de la serie. |
| `SearchResultRow` | Ticker + nombre + botón "+" (44px). Tap en fila → quote; tap en "+" → añade a lista activa con toast "Añadido a Principal". |
| `Toast` | Inferior, sobre bottom bar, auto-dismiss 2.5s. |
| `Skeleton` | Filas fantasma durante carga inicial (no spinners en listas). |

## 4. Pantallas (spec final)

### S0 — Login `/login`
Layout centrado vertical: logo → input email → input password (con toggle visibilidad) → CTA "Entrar" (ancho completo, 48px) → error genérico inline bajo el CTA. Loading state: spinner dentro del botón, inputs deshabilitados. Teclado: `email` type para el primer campo, `returnKeyType=go`.

### S1 — Home / Watchlist `/`
- **Header (48px):** título de lista activa + contador `(12)` a la izquierda; icono ⋮ (editar lista, renombrar, nueva) a la derecha.
- **WatchlistTabs** bajo el header si hay >1 lista.
- **Lista:** `PriceRow` × N, scroll 60fps, pull-to-refresh (snapshot REST), separadores `--border-subtle` 1px. Skeleton ×6 en carga inicial. Empty state: ilustración mínima + "Añade símbolos desde Buscar".
- **ConnectionBadge** flotando abajo-izquierda sobre la bottom bar.
- **BottomTabBar** fija.
- Streaming: suscripción WS a los símbolos visibles de la lista activa; flash on tick en precio y %.

### S2 — Buscar `/search`
Input en header (autofocus al entrar, debounce 300ms, fuzzy contra catálogo). Resultados: `SearchResultRow` × N. Sección "Recientes" (localStorage, máx 5) cuando el input está vacío. Estado sin resultados: "Sin coincidencias para 'XYZ'".

### S3 — Quote Details `/quote/:symbol`
- **Header:** back (←) + ticker + nombre; precio grande mono 24px + cambio absoluto y % en accent-up/down. En vivo vía WS.
- **Mini-chart:** área del día (sparkline con fill), altura 120px, tap → `/chart/:symbol`.
- **Grid stats 2×4:** Apertura · Máx · Mín · Volumen (del snapshot; sin endpoint nuevo).
- **CTA "Ver gráfico →"** ancho completo.
- **Card "En tus listas":** chips con las watchlists que lo contienen; "+" abre sheet para añadir a otra lista.

### S4 — Chart Fullscreen `/chart/:symbol`
- **Header compacto:** back + ticker + precio/% en vivo + ⚙ (config indicadores).
- **TimeframePills:** 5m/1h/1D, persisten por sesión.
- **Chart:** candlestick Lightweight Charts; pinch-zoom, pan, long-press → crosshair con OHLC + valores de indicadores en overlay superior.
- **Subpaneles:** Volumen (default ON, 20% altura), RSI y MACD (default OFF, 25% c/u si activos) apilados bajo precio.
- **IndicatorChips** en fila scrollable bajo el chart.
- Landscape: chart ocupa pantalla completa, header colapsa a overlay.

### S5 — Más `/more`
Lista simple: Tema (toggle oscuro/claro) · Acerca de (versión) · Cerrar sesión (destructivo, confirmación).

## 5. Estados globales (toda pantalla con datos)

| Estado | Comportamiento |
|---|---|
| Stream caído >5s | Badge ámbar "Datos retrasados"; datos congelados (sin flash); retry backoff exponencial 1s→30s |
| Background → foreground | Re-suscripción + snapshot REST; actualización visible < 2s (CA-8) |
| Cambio WiFi↔4G | Reconexión transparente; badge ámbar solo si >5s (CA-9) |
| Token expirado | 401 → redirect `/login` sin mensaje técnico (CA-6) |
| Offline total | Badge rojo; cache del service worker muestra última watchlist en solo lectura |

## 6. Accesibilidad mínima (sin certificar WCAG en MVP)

- Contraste ≥ 4.5:1 en texto sobre bg (los tokens oscuros lo cumplen).
- Verde/rojo nunca como único canal: % siempre con ▲/▼ y signo +/-.
- `aria-label` en iconos de bottom bar y ⋮.

## 7. Entregable visual

La presentación `slides-diseno-pantallas.html` acompaña a este documento: 10 slides con la secuencia de navegación y cada pantalla renderizada en un marco de móvil con estos tokens aplicados.
