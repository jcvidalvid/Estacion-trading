# Estación de Trading — Terminal Móvil de Monitorización

PWA mobile-first de monitorización bursátil (acciones US): watchlists con precios en streaming por WebSocket y gráficos candlestick táctiles con indicadores técnicos. **Sin ejecución de órdenes ni conexión a broker** — el usuario monitoriza y decide aquí, y opera en su broker.

> Una app móvil donde un trader monitoriza sus watchlists en tiempo real y analiza gráficos con indicadores desde una única pantalla fluida.

## Visión

Job-to-be-done: *"Cuando estoy fuera del escritorio, quiero abrir la app, ver de un vistazo mis símbolos en tiempo real y entrar al gráfico de cualquiera con un toque, para decidir en menos de 30 segundos si algo merece mi atención — luego opero en mi broker."*

## Restricción inmutable

Todo el código lo generan agentes de IA a partir de prompts pequeños y verificables. Nada de código manual. Iterar, testear y versionar los prompts es tan importante como versionar el código.

## Alcance del MVP

**Dentro:**

- **Auth:** login email/password + JWT refresh (sin registro self-service, sin 2FA)
- **Watchlists:** crear/editar/eliminar listas, añadir símbolos validados contra catálogo, precios en streaming (last, % cambio, volumen, high/low del día)
- **Gráfico:** candlestick con temporalidades 5m / 1h / 1D e indicadores SMA, EMA, RSI, MACD y Volumen; pinch-to-zoom y pan táctil a 60fps

**Fuera (post-MVP, sin excepciones):** todo lo transaccional (ticket de orden, posiciones, paper trading, conexión a broker), apps nativas, alertas y push (iteración 2), opciones y spreads, contexto de mercado, 2FA, datos nivel II.

## Diseño (modelo IBKR Mobile)

Basado en spec v0.5 + modelo IBKR Mobile aprobado: home = watchlist, quote details, chart fullscreen. Tema oscuro por defecto, bottom bar con 3 tabs (Watchlist / Buscar / Más), mobile-first 360–430px.

**Pantallas:** S0 Login `/login` · S1 Watchlist `/` · S2 Buscar `/search` · S3 Quote Details `/quote/:symbol` · S4 Chart Fullscreen `/chart/:symbol` · S5 Más `/more`

Detalle completo de tokens, componentes y estados en [`DESIGN.md`](./DESIGN.md). Presentación visual en [`slides-diseno-pantallas.html`](./slides-diseno-pantallas.html) (10 slides con cada pantalla renderizada en marco de móvil).

## Arquitectura y stack

- **Frontend:** React 18 + TypeScript + Vite + PWA plugin (service worker, manifest)
- **Backend:** Node.js + Express
- **Datos:** PostgreSQL (usuarios, watchlists) + Redis (pub/sub del bus de datos + cache de snapshots, TTL 60s)
- **Streaming:** una única conexión WebSocket viva en toda la sesión (snapshot inicial + deltas tick-by-tick)
- **Charting:** Lightweight Charts (candidata favorita; decisión final en F1)
- **Proveedor de datos:** pendiente de decisión en F1 (Polygon.io, Finnhub, Alpha Vantage u otro) — la decisión más crítica del proyecto: define coste, cobertura y si los datos son real-time o delayed 15 min

Flujo de datos: `adapter del proveedor → Redis pub/sub → WebSocket server → frontend`

## Flujo de trabajo (4 fases prompt-first)

| Fase | Prompt | Artefacto | Estado |
|---|---|---|---|
| F1 | Investigación: proveedor de datos + charting táctil + capacidades PWA | `RESEARCH.md` | Prompt listo ([`prompt-f1-investigacion.md`](./prompt-f1-investigacion.md)) |
| F2 | Diseño técnico: contratos API, esquema de datos, flujo adapter → Redis → WS → frontend, estrategia PWA | `TECH_DESIGN.md` + `AGENTS.md` | Pendiente |
| F3 | Build A+B: scaffold PWA, login, watchlists + streaming | Código + tests | Pendiente |
| F4 | Build C + hardening + deploy: gráfico táctil, rate limiting, headers, Dockerfile, staging | App navegable | Pendiente |

## Requisitos no funcionales clave

- Latencia de datos: tick < 500ms respecto a fuente; badge "Datos retrasados" si el stream cae >5s, con retry backoff exponencial
- UI: feedback < 100ms; carga inicial < 2s en 4G; 60fps en scroll y gestos del gráfico en gama media (iPhone 12 / Pixel 6)
- El streaming se pausa en background y se reanuda al volver (< 2s, con re-suscripción + snapshot)
- Seguridad: TLS, bcrypt (cost 12), rate limit en login (5 intentos/15min por IP), secrets solo en variables de entorno, errores genéricos
- Offline: lectura de la última watchlist desde cache del service worker

## Estado del proyecto

🚧 **Fase de diseño completada** (31 julio 2026): spec v0.5 lista, diseño visual (DESIGN.md) aprobado, prompt F1 de investigación técnica preparado. Siguiente paso: ejecutar F1 para decidir proveedor de datos, librería de charting y estrategia PWA, y generar `RESEARCH.md`.

## Documentos del repositorio

| Fichero | Contenido |
|---|---|
| `spec-mvp-estacion-trading-v5.pdf` | Spec del MVP v0.5: visión, alcance, requisitos no funcionales, arquitectura, criterios de aceptación CA-1 a CA-10 y Definition of Done |
| `prompt-f1-investigacion.md` | Prompt de Fase 1 listo para pegar en un agente de IA (investigación de proveedor de datos, charting y PWA) |
| `DESIGN.md` | Diseño visual completo: principios, tokens CSS, catálogo de componentes, spec de las 6 pantallas y estados globales |
| `slides-diseno-pantallas.html` | Presentación visual de las pantallas (10 slides, marco de móvil con los tokens aplicados) |
| `README-v0-inicial.md` | Versión inicial del README, archivada el 4 agosto 2026 |
