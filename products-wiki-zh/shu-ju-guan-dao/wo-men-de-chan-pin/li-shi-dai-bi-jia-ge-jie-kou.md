---
description: 获取 SOL 与 SPL 代币的历史价格
---

# 历史代币价格接口

该 Price API 基于已索引的链上数据，支持以 SOL 与 USD 计价获取代币的历史价格。

**基础地址（Base URL）**\
[`https://graphql.astralane.io`](https://graphql.astralane.io)

**Endpoint**\
`GET /api/v1/dataset/price/history?token=<token_address>&from=<start_timestamp>&to=<end_timestamp>`

**参数说明**

* **token：**&#x53;PL 代币的 mint 地址（原生 SOL 可填写 "SOL"）。
* **from：**&#x65F6;间范围起点（Unix 时间戳，单位秒）。
* **to：**&#x65F6;间范围终点（Unix 时间戳，单位秒）。

**注意**

* 单次请求支持的最大时间跨度为 100 分钟。

**描述**\
在指定时间窗口内返回目标代币的历史价格数据，价格同时以 SOL 与 USD 表示。适用于绘图、分析与时间序列研究。

### 查询参数

| 参数      | 类型       | 要求    | 描述                                                         |
| ------- | -------- | ----- | ---------------------------------------------------------- |
| `token` | `string` | ✅ Yes | 要查询历史价格的代币 `token`，填写 SPL 代币的 mint 地址或 "`SOL`"（原生 Solana）。 |
| `from`  | integer  | ✅ Yes | Unix 时间戳（秒）                                                |
| `to`    | integer  |  ✅Yes | Unix 时间戳（秒）                                                |

### 示例请求

```json
GET /api/v1/dataset/price/history?token=So11111111111111111111111111111111111111112&from=1739577600&to=1739577620
```

### 响应示例

```json

[
    {
        "time": 1739577600,
        "value": 199.42621602747087
    }
]

```

