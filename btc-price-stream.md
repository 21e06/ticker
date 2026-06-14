# BTC Price Stream — Exchange WebSocket Reference

Quick reference for public spot/perp WebSocket feeds for BTC price data, for use as alternatives or fallbacks.

## Hyperliquid (primary — used by this app)

**WebSocket URL:**
`wss://api-ui.hyperliquid.xyz/ws`

**Subscribe Messages:**
```json
{ "method": "subscribe", "subscription": { "type": "l2Book", "coin": "BTC", "nSigFigs": 3 } }
{ "method": "subscribe", "subscription": { "type": "activeAssetCtx", "coin": "BTC" } }
{ "method": "subscribe", "subscription": { "type": "trades", "coin": "BTC" } }
```

Each subscription is acknowledged:
```json
{"channel":"subscriptionResponse","data":{"method":"subscribe","subscription":{"type":"l2Book","coin":"BTC","nSigFigs":3,"mantissa":null,"fast":false}}}
```

**Example Response (`l2Book` — order book, `levels[0]` = bids, `levels[1]` = asks):**
```json
{"channel":"l2Book","data":{"coin":"BTC","time":1781416960522,"levels":[[{"px":"64400.0","sz":"3.72067","n":41},{"px":"64300.0","sz":"599.29336","n":543}],[{"px":"64500.0","sz":"2.1","n":12}]]}}
```

**Example Response (`activeAssetCtx` — funding, OI, 24h volume, mark/oracle/mid price):**
```json
{"channel":"activeAssetCtx","data":{"coin":"BTC","ctx":{"funding":"0.0000125","openInterest":"32227.66018","prevDayPx":"63564.0","dayNtlVlm":"1128098483.2000689507","premium":"-0.0002483354","oraclePx":"64429.0","markPx":"64405.0","midPx":"64412.5","impactPxs":["64404.4","64413.0"],"dayBaseVlm":"17577.61508"}}}
```

**Example Response (`trades`):**
```json
{"channel":"trades","data":[{"coin":"BTC","side":"B","px":"64411.0","sz":"0.31377","time":1781416943647,"hash":"0x7f58708caf49d86c80d2043dae4bfc00009388724a4cf73e23211bdf6e4db257","tid":828642708897583,"users":["0xe9bee85af94c8234270523992447e631d27d2146","0xf9109ada2f73c62e9889b45453065f0d99260a2d"]}]}
```

`side` is `"B"` for buy and `"A"` for sell.

**Other useful public channels:**
- `allMids` — best bid/ask midpoint for every asset (`{ "method": "subscribe", "subscription": { "type": "allMids", "dex": "ALL_DEXS" } }`)
- `candle` — OHLCV candles, e.g. `{ "method": "subscribe", "subscription": { "type": "candle", "coin": "BTC", "interval": "5m" } }` →
  ```json
  {"channel":"candle","data":{"t":1781416800000,"T":1781417099999,"s":"BTC","i":"5m","o":"64321.0","c":"64413.0","h":"64413.0","l":"64320.0","v":"126.60117","n":624}}
  ```

Connections must be pinged periodically to stay alive:
```json
{ "method": "ping" }
```

## OKX

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

Trades and liquidations for perps are available on the same connection:
```json
{ "op": "subscribe", "args": [ { "channel": "trades", "instId": "BTC-USDT-SWAP" } ] }
{ "op": "subscribe", "args": [ { "channel": "liquidation-orders", "instType": "SWAP" } ] }
```
```json
{"arg":{"channel":"trades","instId":"BTC-USDT-SWAP"},"data":[{"instId":"BTC-USDT-SWAP","tradeId":"2701364977","px":"64311","sz":"31.27","side":"buy","ts":"1781416584831","count":"8","source":"0","seqId":323647402789}]}
```

## Coinbase

**WebSocket URL:**
`wss://advanced-trade-ws.coinbase.com`

**Subscribe Message:**
```json
{
  "type": "subscribe",
  "channel": "ticker",
  "product_ids": ["BTC-USD"]
}
```

**Example Response:**
```json
{"channel":"ticker","timestamp":"2026-06-14T05:35:45.435813533Z","sequence_num":16,"events":[{"type":"update","tickers":[{"type":"ticker","product_id":"BTC-USD","price":"64281.7","volume_24_h":"3974.98800786","low_24_h":"63454","high_24_h":"64727.27","low_52_w":"59073.01","high_52_w":"126296","price_percent_chg_24_h":"1.21471769193719","best_bid":"64281.7","best_ask":"64281.71","best_bid_quantity":"0.21558849","best_ask_quantity":"0.02287045"}]}]}
```

A `subscriptions` confirmation message follows the first ticker update:
```json
{"channel":"subscriptions","timestamp":"2026-06-14T05:35:45.540672514Z","sequence_num":17,"events":[{"subscriptions":{}}]}
```

Individual trades are available via the `market_trades` channel:
```json
{ "type": "subscribe", "channel": "market_trades", "product_ids": ["BTC-USD"] }
```
```json
{"channel":"market_trades","timestamp":"2026-06-14T05:54:18.66623968Z","sequence_num":0,"events":[{"type":"snapshot","trades":[{"product_id":"BTC-USD","trade_id":"1037798145","price":"64340.74","size":"0.00746872","time":"2026-06-14T05:54:17.799285Z","side":"SELL"}]}]}
```

## Upbit

**WebSocket URL:**
`wss://api.upbit.com/websocket/v1`

**Subscribe Message:**
```json
[
  {"ticket":"test-websocket"},
  {"type":"orderbook","codes":["USDT-BTC"]}
]
```

**Example Response:**
```json
{"type":"orderbook","code":"USDT-BTC","timestamp":1781415569789,"total_ask_size":1.31617837,"total_bid_size":1.89009159,"orderbook_units":[{"ask_price":64464.09,"bid_price":64330.6,"ask_size":0.17332825,"bid_size":0.03833129},{"ask_price":64464.1,"bid_price":64278.97,"ask_size":0.001,"bid_size":0.31746206},{"ask_price":64598.0,"bid_price":64278.96,"ask_size":0.00847249,"bid_size":0.00418192},{"ask_price":64610.63,"bid_price":64278.95,"ask_size":0.022,"bid_size":0.00102},{"ask_price":64613.29,"bid_price":64185.22,"ask_size":0.00240864,"bid_size":0.00186472},{"ask_price":64613.3,"bid_price":64100.0,"ask_size":0.00415487,"bid_size":0.0010},{"ask_price":64658.63,"bid_price":64098.4,"ask_size":0.0007733,"bid_size":0.00239912},{"ask_price":64660.0,"bid_price":64098.39,"ask_size":0.05000000,"bid_size":0.00444662},{"ask_price":64721.33,"bid_price":64072.18,"ask_size":0.639,"bid_size":0.00468221},{"ask_price":64770.1,"bid_price":64038.82,"ask_size":0.00077197,"bid_size":0.002},{"ask_price":64800.0,"bid_price":64033.59,"ask_size":0.01528494,"bid_size":0.00007808},{"ask_price":65000.0,"bid_price":64033.58,"ask_size":0.05116316,"bid_size":0.00627795},{"ask_price":65002.47,"bid_price":64033.56,"ask_size":0.2,"bid_size":0.05327597},{"ask_price":65046.12,"bid_price":64013.73,"ask_size":0.00000785,"bid_size":0.035},{"ask_price":65100.0,"bid_price":64000.0,"ask_size":0.02,"bid_size":0.01903479},{"ask_price":65289.56,"bid_price":63976.51,"ask_size":0.03454433,"bid_size":0.03995655},{"ask_price":65372.16,"bid_price":63927.59,"ask_size":0.00036101,"bid_size":0.4},{"ask_price":65443.91,"bid_price":63900.01,"ask_size":0.02178089,"bid_size":0.00666633},{"ask_price":65488.99,"bid_price":63900.0,"ask_size":0.00000787,"bid_size":0.00000798},{"ask_price":65582.0,"bid_price":63889.5,"ask_size":0.01502183,"bid_size":0.03804081},{"ask_price":65823.42,"bid_price":63829.01,"ask_size":0.05036209,"bid_size":0.00007833},{"ask_price":65951.52,"bid_price":63800.0,"ask_size":0.00490787,"bid_size":0.00000799},{"ask_price":66054.43,"bid_price":63794.07,"ask_size":0.00076483,"bid_size":0.00003135},{"ask_price":66132.96,"bid_price":63752.49,"ask_size":0.00001558,"bid_size":0.00000815},{"ask_price":66157.72,"bid_price":63752.48,"ask_size":0.00000776,"bid_size":0.00003137},{"ask_price":66157.77,"bid_price":63750.0,"ask_size":0.00000776,"bid_size":0.045},{"ask_price":66157.81,"bid_price":63713.15,"ask_size":0.00000776,"bid_size":0.2},{"ask_price":66169.38,"bid_price":63700.0,"ask_size":0.00000776,"bid_size":0.000008},{"ask_price":66400.61,"bid_price":63687.32,"ask_size":0.00000778,"bid_size":0.42},{"ask_price":66584.87,"bid_price":63687.3,"ask_size":0.00000778,"bid_size":0.2492}],"stream_type":"REALTIME","level":0}
```

## Bybit

**WebSocket URL (linear/USDT perps):**
`wss://stream.bybit.com/v5/public/linear`

**WebSocket URL (inverse/coin-margined perps):**
`wss://stream.bybit.com/v5/public/inverse`

**Subscribe Message:**
```json
{
  "op": "subscribe",
  "args": ["publicTrade.BTCUSDT", "allLiquidation.BTCUSDT"]
}
```

**Example Response:**
```json
{"topic":"publicTrade.BTCUSDT","type":"snapshot","ts":1781415763023,"data":[{"T":1781415763021,"s":"BTCUSDT","S":"Sell","v":"0.001","p":"64345.20","L":"MinusTick","i":"35319862-77a2-5ebe-851b-3836573c4fe9","BT":false,"RPI":false,"seq":598180439295},{"T":1781415763021,"s":"BTCUSDT","S":"Sell","v":"0.010","p":"64345.20","L":"ZeroMinusTick","i":"5711a396-555c-5b62-b150-7b105caf871d","BT":false,"RPI":false,"seq":598180439295}]}
```

For inverse perps, subscribe with `BTCUSD` instead of `BTCUSDT`:
```json
{"topic":"publicTrade.BTCUSD","type":"snapshot","ts":1781416510191,"data":[{"T":1781416510190,"s":"BTCUSD","S":"Buy","v":"2485","p":"64318.80","L":"MinusTick","i":"b3a097be-8d22-575a-93dd-4af9c8b8095c","BT":false,"RPI":false,"seq":113240272847}]}
```

Connections must be pinged periodically to stay alive:
```json
{ "op": "ping" }
```

## Binance Futures (USDⓈ-M)

**WebSocket URL (single stream):**
`wss://fstream.binance.com/market/ws`

**WebSocket URL (combined streams):**
`wss://fstream.binance.com/market/stream`

**Subscribe Message:**
```json
{ "method": "SUBSCRIBE", "params": ["btcusdt@aggTrade", "btcusdt@forceOrder"], "id": 2 }
```

**Example Response (trade):**
```json
{"e":"aggTrade","E":1781416439080,"a":3339472008,"s":"BTCUSDT","p":"64339.20","q":"0.123","nq":"0.123","f":7786568797,"l":7786568801,"T":1781416438926,"m":true}
```

**Example Response (liquidation):**
```json
{"e":"forceOrder","E":1781416603268,"o":{"s":"BTCUSDT","S":"BUY","o":"LIMIT","f":"IOC","q":"0.002","p":"64562.50","ap":"64310.60","X":"FILLED","l":"0.002","z":"0.002","T":1781416602260}}
```

On the combined-streams endpoint (`/market/stream`), each message is wrapped: `{"stream":"<name>","data":{...}}`. A useful channel for price tracking is `markPrice`:
```json
{ "method": "SUBSCRIBE", "params": ["btcusdt@markPrice"], "id": 1 }
```
```json
{"stream":"btcusdt@markPrice","data":{"e":"markPriceUpdate","E":1781419401002,"s":"BTCUSDT","p":"64240.38272864","ap":"64240.38272864","P":"64361.80163225","i":"64273.72847826","r":"-0.00006042","T":1781424000000}}
```

## Binance Futures (COIN-M)

**WebSocket URL:**
`wss://dstream.binance.com/ws`

**Subscribe Message:**
```json
{ "method": "SUBSCRIBE", "params": ["btcusd_perp@aggTrade", "btcusd_perp@forceOrder"], "id": 1 }
```

**Example Response (trade):**
```json
{"e":"aggTrade","E":1781416505635,"a":487247786,"s":"BTCUSD_PERP","p":"64304.7","q":"382","f":1137442382,"l":1137442392,"T":1781416505483,"m":true}
```

## Binance Spot

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

## Bitfinex

**WebSocket URL:**
`wss://api-pub.bitfinex.com/ws/2/`

**Subscribe Message:**
```json
{ "event": "subscribe", "channel": "trades", "symbol": "tBTCUSD" }
```

**Example Response:**

Subscription confirmation (returns a numeric `chanId` used to identify subsequent messages on this channel):
```json
{"event":"subscribed","channel":"trades","chanId":856834,"symbol":"tBTCUSD","pair":"BTCUSD"}
```

Trade snapshot — each entry is `[ID, timestamp, amount, price]`, where a negative `amount` is a sell:
```json
[856834,[[1936037547,1781416434609,0.00000064,64256],[1936037542,1781416425843,0.01352295,64255],[1936037537,1781416422248,-0.0002,64243]]]
```

## BitMEX

**WebSocket URL:**
`wss://www.bitmex.com/realtime`

**Subscribe Message:**
```json
{ "op": "subscribe", "args": ["trade:XBTUSD", "liquidation:XBTUSD"] }
```

**Example Response:**
```json
{"table":"trade","action":"partial","keys":[],"types":{"timestamp":"timestamp","symbol":"symbol","side":"symbol","size":"long","price":"float","tickDirection":"symbol","trdMatchID":"guid","grossValue":"long","homeNotional":"float","foreignNotional":"float","trdType":"symbol","pool":"symbol"},"filter":{"pool":"Aggregated","symbol":"XBTUSD"},"data":[{"timestamp":"2026-06-14T05:52:41.303Z","symbol":"XBTUSD","side":"Sell","size":200,"price":64330.3,"tickDirection":"ZeroMinusTick","trdMatchID":"00000000-0000-0000-0000-000000000000","grossValue":31100000,"homeNotional":0.00311,"foreignNotional":200,"trdType":"Regular","pool":"Aggregated"}]}
```

## Deribit

**WebSocket URL:**
`wss://www.deribit.com/ws/api/v2`

**Subscribe Message:**
```json
{ "method": "public/subscribe", "params": { "channels": ["trades.BTC-PERPETUAL.100ms"] } }
```

**Example Response:**
```json
{"jsonrpc":"2.0","method":"subscription","params":{"channel":"trades.BTC-PERPETUAL.100ms","data":[{"timestamp":1781416501150,"price":64345.0,"amount":30.0,"direction":"buy","index_price":64347.47,"instrument_name":"BTC-PERPETUAL","trade_seq":291801232,"mark_price":64345.22,"tick_direction":2,"contracts":3.0,"trade_id":"433949015"}]}}
```

Connections must be pinged periodically to stay alive (every ~30s):
```json
{ "method": "public/ping" }
```

## Bitstamp

**WebSocket URL:**
`wss://ws.bitstamp.net/`

**Subscribe Message:**
```json
{ "event": "bts:subscribe", "data": { "channel": "live_trades_btcusd" } }
```

**Example Response:**
```json
{"data":{"id":590708212,"timestamp":"1781416599","amount":0.00024876,"amount_str":"0.00024876","price":64319.29,"price_str":"64319.29","type":0,"microtimestamp":"1781416599325000","buy_order_id":2017614334640128,"sell_order_id":2017614282440706},"channel":"live_trades_btcusd","event":"trade"}
```

`type` is `0` for buy and `1` for sell.

