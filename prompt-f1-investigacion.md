# Prompt F1 — Investigación Técnica (Terminal Móvil de Monitorización)

> **Uso:** copia todo el contenido del bloque inferior y pégalo en tu agente de IA (Cursor, Claude, etc.) como primer mensaje. Adjunta `spec-mvp-estacion-trading-v5.md` si el agente lo soporta.
> **Output esperado:** `RESEARCH.md` de 1–2 páginas con la tabla de decisión rellena y una recomendación final por área.

---

## PROMPT (copiar desde aquí)

**Rol:** Investigador técnico senior especializado en fintech y datos de mercado en tiempo real.

**Contexto:** Estoy construyendo una PWA mobile-first de monitorización bursátil (acciones US): watchlists con precios en streaming por WebSocket y gráficos candlestick táctiles con indicadores. NO hay ejecución de órdenes ni conexión a broker. Todo el código posterior lo generarán agentes de IA. Adjunto la spec completa como referencia.

**Tarea:** Investiga y decide las 3 áreas bloqueantes del proyecto. Tu output será la única entrada de la fase de diseño técnico (F2), así que cada recomendación debe ser accionable, con alternativas descartadas y motivo del descarte.

### Área 1 — Proveedor de datos de mercado

Necesito precios de acciones US con estas restricciones:
- WebSocket (no polling REST) para streaming tick-by-tick.
- Snapshot inicial + deltas: `{ symbol, last, change, changePercent, volume, high, low, timestamp }`.
- Suscripciones simultáneas para ~50 símbolos por usuario (watchlist).
- Coste: gratis o < $50/mes para uso personal/MVP.
- REST adicional para histórico OHLCV (necesario para los gráficos 5m, 1h, 1D).

Compara al menos: Polygon.io, Finnhub, Alpha Vantage, Twelve Data, y cualquier otra que encaje mejor. Para cada una: cobertura, si los datos son **real-time o delayed 15 min** (crítico: los planes gratis suelen ser delayed), límites de suscripciones WebSocket, límites de rate en REST, licencia de uso (¿puedo mostrar datos en una app propia?), y coste real al salir del free tier.

**Decisión requerida:** ¿acepto datos delayed 15 min gratis para el MVP, o pago real-time desde el día 1? Argumenta con el caso de uso: el usuario monitoriza y decide, pero opera en su broker.

### Área 2 — Librería de charting táctil

Requisitos duros:
- Candlestick + indicadores SMA, EMA, RSI, MACD, Volumen.
- Gestos táctiles: pinch-to-zoom, pan — 60fps en gama media (iPhone 12 / Pixel 6).
- Ligera (bundle pequeño, es una PWA móvil en 4G).
- Licencia compatible con app propia (comercial futura).

Compara al menos: Lightweight Charts (TradingView), TradingView Charting Library, Apache ECharts, uPlot. Para cada una: peso del bundle, rendimiento móvil conocido, soporte de gestos táctiles nativo, esfuerzo para añadir los 5 indicadores (¿vienen incluidos o se calculan aparte?), y licencia/coste.

**Decisión requerida:** librería recomendada + si los indicadores se calculan en frontend (¿con qué librería: technicalindicators, tulind?) o vienen del proveedor de datos.

### Área 3 — Capacidades y límites PWA (2026)

Verifica el estado actual de:
- Instalabilidad en home screen: flujo en iOS Safari vs Android Chrome (¿sigue iOS sin prompt automático de instalación?).
- Service worker: cache de la última watchlist para lectura offline.
- WebSocket: comportamiento real al pasar la app a background en iOS y Android (¿se congela? ¿se mata? ¿cuánto tarda?). Estrategia recomendada de pausa/reanudación.
- Limitaciones conocidas que afecten a: streaming continuo, almacenamiento de tokens seguro, rendimiento de canvas en iOS.
- Notificaciones push en PWA-iOS: estado actual y limitaciones (no es MVP, pero condiciona la iteración 2 — solo 1 párrafo).

### Formato del output (RESEARCH.md)

```
# RESEARCH.md — F1 Terminal Móvil de Monitorización
## Decisión 1: Proveedor de datos
| Candidato | Real-time/Delayed | WS símbolos | REST OHLCV | Coste MVP | Licencia display |
...
**RECOMENDADO:** X — porque...
**Descartados:** Y (motivo), Z (motivo)
## Decisión 2: Charting
(misma tabla)
## Decisión 3: PWA
- Instalación iOS/Android: ...
- WebSocket background: ...
- Estrategia recomendada: ...
## Riesgos abiertos
(lista de lo que no se pudo verificar y requiere prueba empírica en F3/F4)
```

**Constraints:** no escribas código; no asumas precios sin marcarlos como "verificar en la web del proveedor"; si un dato es incierto, márcalo `[VERIFICAR]` en vez de inventarlo. Máximo 2 páginas.

---

## Notas de supervisión humana (no van en el prompt)

- La decisión 1 (real-time vs delayed) es la única que te recomiendo validar tú mismo antes de F2: delayed 15 min es aceptable para monitorizar, pero si tu uso real es intradía en apertura, el salto a Polygon Starter (~$29/mes) puede ser inevitable.
- Cuando el agente entregue `RESEARCH.md`, revisa que ningún precio/límite esté marcado como seguro sin fuente: los planes de datos cambian a menudo.
