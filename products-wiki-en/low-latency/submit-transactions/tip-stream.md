---
icon: money-bills
---

# Tip Stream

Live percentile snapshot of tip sizes paid by transactions **landed through Astralane**. Useful for picking a tip that is competitive enough to get your transaction landed through Astralane without over-paying.

### What it provides

A rolling snapshot with five percentile buckets of the tip amounts (in **lamports**) paid by transactions that Astralane **successfully landed** in the last 15 minutes:

* `landed_tips_25th_percentile` - 25% of landed tips were at or below this (not recommeded)
* `landed_tips_50th_percentile` - the median landed tip
* `landed_tips_75th_percentile`
* `landed_tips_95th_percentile`
* `landed_tips_99th_percentile`

If you are sending a transaction, compare your intended tip against these buckets. A tip around the 50th percentile generally lands; a tip in the 95th percentile is what you pay during contention. Anything below the 25th is likely too low to land.

#### REST - single snapshot

`GET /tip_floor` returns a JSON array with one element: the current snapshot.

```bash
curl -s https://tips.astralane.io/tip_floor | jq
```

Example response:

```json
[
  {
    "time": "2026-04-22T11:22:52Z",
    "landed_tips_25th_percentile": 10000,
    "landed_tips_50th_percentile": 21000,
    "landed_tips_75th_percentile": 77952,
    "landed_tips_95th_percentile": 1000000,
    "landed_tips_99th_percentile": 4759970
  }
]
```

#### WebSocket - live stream

`GET /tip_stream` upgrades to a WebSocket. The server pushes the current snapshot immediately on connect, then pushes a fresh snapshot after every 15-minute refresh. Payload shape is identical to the REST response.

```bash
websocat wss://tips.astralane.io/tip_stream
```

### Rate limits

* **REST**: 10 requests per second per client IP.
* **WebSocket**: 2 concurrent connections per client IP.

Exceeding either limit returns HTTP `429 Too Many Requests`.
