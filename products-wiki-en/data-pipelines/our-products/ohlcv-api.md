---
description: ohlcv data of pools
---

# OHLCV API

The **OHLCV API** provides comprehensive candlestick data (Open, High, Low, Close, Volume) for Solana token pools, delivering both real-time and historical market insights. Designed with developers in mind, this API facilitates seamless integration into trading tools, analytics platforms, and diverse financial applications.

With flexible time intervals ranging from minute-by-minute granularity to daily summaries, the OHLCV API empowers precise and tailored market analysis. Whether you’re creating advanced charting solutions, automated trading bots, or performing in-depth market research, this API ensures you always have access to reliable, accurate, and timely market data.



**Base URL**

[https://graphql.astralane.io/api/v1/dataset/trade/ohlcv](https://graphql.astralane.io/api/v1/dataset/trade/ohlcv)

### GET/<mark style="color:red;">`{pool_address}`</mark>?interval=<mark style="color:red;">`{interval}`</mark>\&from=<mark style="color:red;">`{timestamp}`</mark>\&to=<mark style="color:red;">`{timestamp}`</mark>

### Query Parameters

| Parameter  | Type   | Description                                                                                      |
| ---------- | ------ | ------------------------------------------------------------------------------------------------ |
| `interval` | string | The interval for OHLCV data <mark style="color:red;">`(e.g., 1s, 1m, 5m, 15m, 1h, etc.).`</mark> |
| `pool`     | string | The pool address to fetch OHLCV data for.                                                        |
| `from`     | number | (Optional) Start timestamp in UNIX format (seconds).                                             |
| `to`       | number | (Optional) End timestamp in UNIX format (seconds).                                               |

### Supported Intervals

| Shorthand | Interval   |
| --------- | ---------- |
| 1s        | 1 SECOND   |
| 5s        | 5 SECONDS  |
| 15s       | 15 SECONDS |
| 1m        | 1 MINUTE   |
| 3m        | 3 MINUTES  |
| 5m        | 5 MINUTES  |
| 15m       | 15 MINUTES |
| 30m       | 30 MINUTES |
| 1H        | 1 HOUR     |
| 4H        | 4 HOURS    |
| 6H        | 6 HOURS    |
| 8H        | 8 HOURS    |
| 12H       | 12 HOURS   |
| 1D        | 1 DAY      |

{% hint style="warning" %}
Params are case-senstitive
{% endhint %}

### Response

| Parameter   | Type   | Description                              |
| ----------- | ------ | ---------------------------------------- |
| `time`      | number | UNIX timestamp (seconds).                |
| `open`      | number | Candel open price in quote token.        |
| `high`      | number | The highest candel price in quote token. |
| `low`       | number | The lowest candel price in quote token.  |
| `close`     | number | Candel close price in quote token.       |
| `volume`    | number | Candel volume in quote token.            |
| `volue_usd` | number | Candel volume in USD.                    |

### **Sample Request**

```javascript
GET /api/v1/dataset/trade/ohlcv/58oQChx4yWmvKdwLLZzBi4ChoCc2fqCUWBkwMihLYQo2?interval=1m&from=1736790000&to=1736793600
```

<pre><code><strong>header:{
</strong>x-api-key:&#x3C;your_api_key>
}
</code></pre>

```url
curl --location '<https://graphql.astralane.io/api/v1/dataset/trade/ohlcv/GRhDDqU6bYU9xHam5AcZPvrydp3JsjjJVxgoVXZphqGo?interval=1s>' \\
--header 'x-api-key: <your_api_key>'
```

### **Response**

```json
[
  {
    "time": 1736791800,
    "open": 177.88,
    "high": 178.12,
    "low": 177.76,
    "close": 178.00,
    "volume": 123.45,
    "volume_usd": 22000.00
  },
  {
    "time": 1736791860,
    "open": 178.00,
    "high": 178.25,
    "low": 177.95,
    "close": 178.15,
    "volume": 150.30,
    "volume_usd": 27000.50
  }
]

```
