# SPEC MVP v0.5 — Terminal Móvil de Análisis y Monitorización (Prompt-First)

**Fecha:** 31 julio 2026 | **Estado:** Listo para Fase 1 | **Objetivo:** MVP **mobile-first PWA** de análisis en tiempo real, **sin ejecución de órdenes**

## 1. Visión (1 frase)

Una app móvil donde un trader monitoriza sus watchlists en tiempo real y analiza gráficos con indicadores técnicos desde una única pantalla fluida — la ejecución se queda en su broker actual.

**Cambios de rumbo acumulados:**
- **v0.3:** se elimina todo el flujo transaccional (ticket de orden, posiciones, paper trading). Terminal de análisis y monitorización.
- **v0.4:** la plataforma principal pasa a ser **móvil**, implementada como **PWA** (una sola codebase, instalable en home screen, sin app stores).
- **v0.5:** se define la **estructura de pantallas**: login → pantalla principal tipo dashboard con ventanas (una por funcionalidad) → cada ventana abre su pantalla completa de detalle.

**Restricción inmutable:** todo el código lo generan agentes de IA a partir de prompts pequeños y verificables. Nada de código manual.

## 2. Estructura de pantallas (nueva)

La app tiene un modelo de navegación simple de 2 niveles: **dashboard con ventanas → pantalla completa**.

### Nivel 0 — Login
- Pantalla única: email + password + botón. Estado de carga y errores genéricos. Sin registro self-service en el MVP (alta manual o seed).

### Nivel 1 — Pantalla principal (Dashboard)
Pantalla principal tras el login, organizada en **ventanas (cards/tiles) apiladas verticalmente**, una por funcionalidad:

- **Ventana Watchlist:** muestra la watchlist activa con sus símbolos — cada fila con símbolo, último precio y % cambio (verde/rojo), actualizándose en tiempo real. Header con nombre de la lista y contador.
- **Ventana Gráfico rápido:** miniatura del gráfico del símbolo seleccionado (por defecto el primero de la watchlist, o el último visitado) con temporalidad actual.

Reglas de interacción:
- Al **tocar una ventana**, se abre la **pantalla completa** de esa funcionalidad (Nivel 2).
- Tocar un **símbolo concreto** dentro de la ventana watchlist abre directamente el gráfico completo de ese símbolo (atajo).
- Volver atrás (gesto swipe desde el borde o botón back) regresa siempre al dashboard.
- El dashboard mantiene **estado vivo**: los precios de la ventana watchlist y la miniatura del gráfico se actualizan en tiempo real, no son estáticos.

### Nivel 2 — Pantallas completas
- **Watchlist completa:** lista completa de símbolos con todas las columnas (last, % cambio, volumen, high/low). Selector de watchlists (swipe horizontal o tabs), añadir/eliminar símbolos, crear/renombrar/eliminar listas. Pull-to-refresh.
- **Gráfico completo:** candlestick a pantalla completa con selector de temporalidad (5m, 1h, 1D), toggle de indicadores (SMA, EMA, RSI, MACD, Volumen), pinch-to-zoom y pan táctil. Header con símbolo, último precio y % cambio.

## 3. Alcance del MVP

### Dentro (Must)

| # | Módulo | Comportamiento mínimo |
|---|--------|----------------------|
| A | Auth | Login email/password + JWT refresh. Sin 2FA en MVP. |
| B | Watchlists | Crear/editar/eliminar watchlists, añadir símbolos validados contra catálogo, precios por WebSocket (last, % cambio, volumen, high/low del día). |
| C | Gráfico | Candlestick con 3 temporalidades (5m, 1h, 1D) + indicadores SMA, EMA, RSI, MACD, Volumen. Sin dibujo, sin overlay. |

### Requisitos de experiencia móvil (transversales)

- **Mobile-first:** diseño y desarrollo pensados primero para pantalla de 360–430px; desktop es escala-up, no prioridad.
- **PWA:** instalable en home screen (manifest + service worker), splash screen, icono propio, fullscreen sin barra del navegador.
- **Touch-first:** targets táctiles ≥ 44px; pull-to-refresh en listas; gráfico con pinch-to-zoom y pan; swipe-back para volver al dashboard; sin interacciones que dependan de hover.
- **Navegación:** dashboard como pantalla única de entrada; cada funcionalidad se abre a pantalla completa; back siempre al dashboard. Transiciones nativas-like (slide).
- **Tipografía:** mínimo 14pt escalable, legible en exteriores.

### Fuera (post-MVP, sin excepciones)

- **Todo lo transaccional:** ticket de orden, posiciones, balance, paper trading, conexión a broker
- Apps nativas iOS/Android (la PWA las sustituye en el MVP)
- Alertas y notificaciones push (se evalúan en iteración 2)
- Opciones y spreads
- Contexto de mercado (earnings, noticias, hotlists)
- 2FA, datos nivel II, layout multi-panel en landscape, reordenar/redimensionar ventanas del dashboard

> **Nota:** alertas de precio es el primer candidato de la iteración 2 — encaja con un producto de monitorización móvil y añadiría una tercera ventana al dashboard.

## 4. Usuario objetivo (1 persona)

**Trader activo que monitoriza el mercado desde el móvil.** Job-to-be-done: "Cuando estoy fuera del escritorio, quiero abrir la app, ver de un vistazo mis símbolos en tiempo real y entrar al gráfico de cualquiera con un toque, para decidir en menos de 30 segundos si algo merece mi atención — luego opero en mi broker."

## 5. Requisitos no funcionales (solo los que importan)

- **Latencia datos:** tick < 500ms respecto a fuente; reconexión con backoff exponencial y badge "Datos retrasados" si el stream cae; el streaming debe **sobrevivir a cambios de red** (WiFi ↔ 4G) y a la vuelta de background (re-suscripción al volver al foreground).
- **UI:** feedback < 100ms; tema claro/oscuro; carga inicial < 2s en 4G; 60fps en scroll y zoom/pan del gráfico en gama media (iPhone 12 / Pixel 6). Las transiciones dashboard ↔ pantalla completa a 60fps.
- **Consumo:** el streaming se pausa cuando la app pasa a background (ahorro de batería y datos); reanudación instantánea al volver.
- **Seguridad:** TLS, bcrypt (cost 12), rate limit en login (5 intentos/15min por IP), secrets solo en variables de entorno, errores genéricos que no revelen si un email existe. Tokens en storage seguro de la PWA.
- **Fiabilidad:** si falla el streaming, mostrar último snapshot con indicador de desconexión. Lectura offline de la última watchlist cargada (cache del service worker).

Se eliminan respecto a v0.1: audit trail de órdenes, compliance PCI-DSS, WCAG AA formal, 99.9% uptime.

## 6. Arquitectura y datos (decisiones clave para F1/F2)

- **Stack sugerido:** React 18 + TypeScript + Vite + PWA plugin (service worker, manifest); Node.js + Express; PostgreSQL (usuarios, watchlists); Redis (pub/sub del bus de datos + cache de snapshots, TTL 60s); WebSocket para streaming.
- **Routing:** `/login`, `/` (dashboard), `/watchlist`, `/chart/:symbol`. Estado de la suscripción WebSocket compartido entre rutas (una única conexión viva en toda la sesión, no una por pantalla).
- **Librería de charting:** debe rendir a 60fps en móvil y soportar gestos táctiles — candidata favorita: Lightweight Charts. Decisión final en F1.
- **Modelo de datos mínimo:** `users`, `watchlists`, `watchlist_symbols`, `instruments` (catálogo maestro). Sin tablas de órdenes, posiciones ni cuentas.
- **Flujo de datos:** adapter del proveedor → Redis pub/sub → WebSocket server → frontend. Snapshot inicial + deltas tick-by-tick: `{ symbol, last, change, changePercent, volume, high, low, timestamp }`.
- **Proveedor de datos:** pendiente de decidir en F1 (Polygon.io, Finnhub, Alpha Vantage u otro). Decisión más crítica del proyecto: define coste, cobertura y si los datos son real-time o delayed 15 min.

## 7. Flujo de prompts (4 fases)

| Fase | Prompt | Artefacto |
|------|--------|-----------|
| F1 | Investigación: proveedor de datos (WebSocket, acciones US, gratis/barato) + librería de charting táctil + capacidades PWA | RESEARCH.md |
| F2 | Diseño técnico: contratos API, esquema de datos, flujo adapter → Redis → WS → frontend, estrategia PWA, modelo de navegación dashboard → detalle | TECH_DESIGN.md + AGENTS.md |
| F3 | Build A+B: scaffold PWA mobile-first, login, dashboard con ventanas, watchlists + streaming | Código + tests |
| F4 | Build C + hardening + deploy: gráfico táctil completo, rate limiting, headers, Dockerfile, staging | App navegable |

Cada prompt de build sigue el formato de la spec v0.1: rol, spec de comportamiento, ejemplos de I/O (feliz/edge/error), constraints, tests obligatorios.

## 8. Criterios de aceptación del MVP

- **CA-1:** Dado un login válido, cuando el usuario entra, ve el dashboard con la ventana watchlist mostrando precios actualizándose en < 500ms.
- **CA-2:** Dada una watchlist con 50 símbolos, cuando el usuario abre su pantalla completa, el sistema mantiene una única conexión de streaming para todos, con scroll fluido.
- **CA-3:** Dado un gráfico 1h, cuando el usuario añade RSI (14 periodos), se calcula y renderiza sin bloquear la UI; pinch-to-zoom y pan responden a 60fps en gama media.
- **CA-4 (Navegación):** Dado el dashboard, cuando el usuario toca la ventana watchlist, se abre la pantalla completa de watchlist; cuando toca un símbolo concreto, se abre el gráfico completo de ese símbolo; al hacer back, vuelve al dashboard.
- **CA-5:** Dada una caída del WebSocket (>5s sin heartbeat), la UI muestra el badge de desconexión y reintenta con backoff exponencial.
- **CA-6:** Dado un token expirado, cuando el usuario hace una acción protegida, el sistema devuelve 401 y redirige a login sin exponer datos.
- **CA-7:** Dado el tema oscuro activo, cuando el usuario navega a cualquier pantalla, todos los componentes respetan el tema sin flickers.
- **CA-8 (Móvil):** Dado que la app pasa a background y vuelve al foreground, el streaming se reanuda automáticamente y muestra un snapshot actualizado en < 2s.
- **CA-9 (Móvil):** Dado un cambio de red WiFi → 4G en curso, la conexión se restablece sin intervención del usuario.
- **CA-10 (PWA):** Dado un usuario en su móvil, la app es instalable en home screen y arranca fullscreen con icono propio.

## 9. Definition of Done

- Código compila, pasa lint + type-check (TS strict), sin secrets hardcodeados.
- Cada módulo: 3 tests unitarios (feliz/edge/error) + 1 de integración.
- CA-1 a CA-10 verificados en staging, incluidos CA-8/CA-9/CA-10 en un dispositivo físico real (no solo emulador).
- Lighthouse PWA check en verde (instalable, service worker activo).
- Prompts y outputs versionados en Git junto al código.

**Regla de oro (sin cambios):** un prompt pequeño y bien especificado produce mejor código que un monólogo gigante. Iterar, testear y versionar los prompts es tan importante como versionar el código.
