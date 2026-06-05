---
description: 用于获取池子的 K 线数据（Open/High/Low/Close/Volume）
---

# K线OHLCV接口

**K线OHLCV接口**提供 Solana 代币池的完整蜡烛图数据（开盘、最高、最低、收盘、成交量），覆盖实时与历史视角。该接口面向开发者设计，便于无缝集成至交易工具、分析平台与各类金融应用。

该接口支持从秒级到日级的灵活时间粒度，便于进行精细化与定制化的市场分析。无论是构建高级图表、自动化交易机器人，还是开展深度研究，都可获得可靠、准确、及时的市场数据。

**基础地址（Base URL）**

[https://graphql.astralane.io/api/v1/dataset/trade/ohlcv](https://graphql.astralane.io/api/v1/dataset/trade/ohlcv)

### GET/<mark style="color:red;">`{pool_address}`</mark>?interval=<mark style="color:red;">`{interval}`</mark>\&from=<mark style="color:red;">`{timestamp}`</mark>\&to=<mark style="color:red;">`{timestamp}`</mark>

### 查询参数（Query Parameters）

| 参数         | 类型     | 描述                                                                          |
| ---------- | ------ | --------------------------------------------------------------------------- |
| `interval` | string | K 线时间粒度<mark style="color:red;">`(e.g., 1s, 1m, 5m, 15m, 1h, etc.).`</mark> |
| `pool`     | string | 目标池子的地址。                                                                    |
| `from`     | number | 起始时间戳（UNIX 秒）。                                                              |
| `to`       | number | 结束时间戳（UNIX 秒）。                                                              |

### 支持的时间粒度（Supported Intervals）

| 简写  | 粒度         |
| --- | ---------- |
| 1s  | 1 SECOND   |
| 5s  | 5 SECONDS  |
| 15s | 15 SECONDS |
| 1m  | 1 MINUTE   |
| 3m  | 3 MINUTES  |
| 5m  | 5 MINUTES  |
| 15m | 15 MINUTES |
| 30m | 30 MINUTES |
| 1H  | 1 HOUR     |
| 4H  | 4 HOURS    |
| 6H  | 6 HOURS    |
| 8H  | 8 HOURS    |
| 12H | 12 HOURS   |
| 1D  | 1 DAY      |

{% hint style="warning" %}
参数区分大小写（Params are case-sensitive）。
{% endhint %}

### 响应字段

| 参数          | 类型     | 描述                  |
| ----------- | ------ | ------------------- |
| `time`      | number | UNIX 时间戳 (seconds). |
| `open`      | number | 该周期的开盘价（以计价代币计价）。   |
| `high`      | number | 该周期的最高价（以计价代币计价）。   |
| `low`       | number | 该周期的最低价（以计价代币计价）。   |
| `close`     | number | 该周期的收盘价（以计价代币计价）。   |
| `volume`    | number | 该周期的成交量（以计价代币计量）。   |
| `volue_usd` | number | 该周期的成交量（USD 计价）。    |

### 示例请求

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

### 响应示例

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
