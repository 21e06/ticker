# BTC Price Stream — Exchange WebSocket Reference

Reference for the public spot/perp WebSocket feeds used by this app to track BTC price across multiple sources. Each source connects via WebSocket for live price updates and (where available) a REST endpoint to fetch the clock-aligned open price for the current 5-minute window.

## Sources used by this app

### Hyperliquid

**WebSocket URL:**
`wss://api-ui.hyperliquid.xyz/ws`

**Subscribe Message:**
```json
{ "method": "subscribe", "subscription": { "type": "activeAssetCtx", "coin": "BTC" } }
```

**Example Response (mark price used as price source):**
```json
{"channel":"activeAssetCtx","data":{"coin":"BTC","ctx":{"funding":"0.0000125","openInterest":"32227.66018","prevDayPx":"63564.0","dayNtlVlm":"1128098483.2000689507","premium":"-0.0002483354","oraclePx":"64429.0","markPx":"64405.0","midPx":"64412.5","impactPxs":["64404.4","64413.0"],"dayBaseVlm":"17577.61508"}}}
```
Price is read from `data.ctx.markPx`.

**Open price (REST, `candleSnapshot`):**
```
POST https://api.hyperliquid.xyz/info
{ "type": "candleSnapshot", "req": { "coin": "BTC", "interval": "5m", "startTime": <windowStart>, "endTime": <now> } }
```
Response is an array of candles; open price is `data[0].o`.

Connection must be pinged periodically to stay alive:
```json
{ "method": "ping" }
```

### OKX

**WebSocket URL:**
`wss://ws.okx.com:8443/ws/v5/public`

**Subscribe Message:**
```json
{ "op": "subscribe", "args": [ { "channel": "tickers", "instId": "BTC-USDT" } ] }
```

**Example Response:**
```json
{"arg":{"channel":"tickers","instId":"BTC-USDT"},"data":[{"instType":"SPOT","instId":"BTC-USDT","last":"64312.9","lastSz":"0.00001408","askPx":"64312.9","askSz":"1.34197806","bidPx":"64312.8","bidSz":"3.32864302","open24h":"63591.6","high24h":"64758.8","low24h":"63493.1","sodUtc0":"64454.4","sodUtc8":"64305.5","volCcy24h":"396271423.832180677","vol24h":"6175.12222953","ts":"1781414919074"}]}
```
Price is read from `data[0].last`.

**Open price (REST):**
```
GET https://www.okx.com/api/v5/market/candles?instId=BTC-USDT&bar=5m&limit=1
```
Open price is `data.data[0][1]`.

### Coinbase

**WebSocket URL:**
`wss://advanced-trade-ws.coinbase.com`

**Subscribe Message:**
```json
{ "type": "subscribe", "channel": "ticker", "product_ids": ["BTC-USD"] }
```

**Example Response:**
```json
{"channel":"ticker","timestamp":"2026-06-14T05:35:45.435813533Z","sequence_num":16,"events":[{"type":"update","tickers":[{"type":"ticker","product_id":"BTC-USD","price":"64281.7","volume_24_h":"3974.98800786","low_24_h":"63454","high_24_h":"64727.27","low_52_w":"59073.01","high_52_w":"126296","price_percent_chg_24_h":"1.21471769193719","best_bid":"64281.7","best_ask":"64281.71","best_bid_quantity":"0.21558849","best_ask_quantity":"0.02287045"}]}]}
```
Price is read from each `events[].tickers[].price`.

**Open price (REST):**
```
GET https://api.exchange.coinbase.com/products/BTC-USD/candles?granularity=300&limit=1
```
Response rows are `[time, low, high, open, close, volume]`; open price is `data[0][3]`.

### Binance Spot

**WebSocket URL:**
`wss://data-stream.binance.vision:9443/ws`

**Subscribe Message:**
```json
{ "method": "SUBSCRIBE", "params": ["btcusdt@aggTrade"], "id": 1 }
```

**Example Response:**
```json
{"e":"aggTrade","E":1781416576889,"s":"BTCUSDT","a":3988316038,"p":"64348.95000000","q":"0.00009000","f":6407186253,"l":6407186253,"T":1781416576889,"m":true,"M":true}
```
Price is read from `p`.

**Open price (REST):**
```
GET https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=5m&limit=1
```
Response rows are `[openTime, open, high, low, close, ...]`; open price is `data[0][1]`.

### Binance Futures (USDⓈ-M)

**WebSocket URL (combined streams):**
`wss://fstream.binance.com/market/stream`

**Subscribe Message:**
```json
{ "method": "SUBSCRIBE", "params": ["btcusdt@markPrice"], "id": 1 }
```

**Example Response:**
```json
{"stream":"btcusdt@markPrice","data":{"e":"markPriceUpdate","E":1781419401002,"s":"BTCUSDT","p":"64240.38272864","ap":"64240.38272864","P":"64361.80163225","i":"64273.72847826","r":"-0.00006042","T":1781424000000}}
```
Price is read from `data.p`.

**Open price (REST):**
```
GET https://fapi.binance.com/fapi/v1/klines?symbol=BTCUSDT&interval=5m&limit=1
```
Response rows are `[openTime, open, high, low, close, ...]`; open price is `data[0][1]`.

### Bybit

**WebSocket URL (linear/USDT perps):**
`wss://stream.bybit.com/v5/public/linear`

**Subscribe Message:**
```json
{ "op": "subscribe", "args": ["publicTrade.BTCUSDT"] }
```

**Example Response:**
```json
{"topic":"publicTrade.BTCUSDT","type":"snapshot","ts":1781415763023,"data":[{"T":1781415763021,"s":"BTCUSDT","S":"Sell","v":"0.001","p":"64345.20","L":"MinusTick","i":"35319862-77a2-5ebe-851b-3836573c4fe9","BT":false,"RPI":false,"seq":598180439295}]}
```
Price is read from the last entry in `data[]`, field `p`.

**Open price (REST):**
```
GET https://api.bybit.com/v5/market/kline?category=linear&symbol=BTCUSDT&interval=5&limit=1
```
Response rows are `[start, open, high, low, close, volume, turnover]`; open price is `result.list[0][1]`.

Connection must be pinged periodically to stay alive:
```json
{ "op": "ping" }
```

### Polymarket

**WebSocket URL:**
`wss://ws-live-data.polymarket.com/`

**Subscribe Message:**
```json
{ "action": "subscribe", "subscriptions": [ { "topic": "crypto_prices_chainlink", "type": "update", "filters": "{\"symbol\":\"btc/usd\"}" } ] }
```

**Example Response (live update):**
```json
{"topic":"crypto_prices_chainlink","type":"update","payload":{"value":64345.0}}
```
Price is read from `payload.value`.

**Example Response (initial backlog):**
```json
{"payload":{"data":[{"timestamp":1781416800000,"value":64321.0}, ...]}}
```
On the initial backlog, the entry with the latest `timestamp <= windowStart` is used as the baseline (open) price. There is no separate REST endpoint for this source.

## Notes

- All sources report price in USD except where noted (Coinbase, OKX, Binance, Bybit quote in USDT/USD pairs which are treated as ~1:1 with USD).
- "Open price" for each source is the price at the start of the current clock-aligned 5-minute window, used to compute that source's up/down/flat direction.
- If a source's REST open-price fetch fails or is unavailable (Polymarket), the first received live price is used as the baseline instead.
