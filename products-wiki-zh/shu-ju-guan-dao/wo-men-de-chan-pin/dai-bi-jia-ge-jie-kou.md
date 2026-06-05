---
description: 获取 SOL 与受支持的 SPL 代币的最新市场价格。
---

# 代币价格接口

代币价格接口允许用户基于最新可用的市场数据，实时获取代币以 SOL 与 USD 计价的价格。

**基础地址（Base URL）**\
[https://graphql.astralane.io](https://graphql.astralane.io)

**Endpoint**\
`GET /api/v1/price-by-token?tokens=token1,token2,token3`

**描述：**\
通过在 `tokens` 查询参数中提供以逗号分隔的代币 mint 地址列表（最多 50 个），返回这些代币的最新价格。价格同时以 SOL 与 USD 返回。

**注意：**&#x5355;次请求最多支持查询 50 个代币 mint 地址。

### 查询参数

| 参数       | 类型       | 必要性   | 描述                            |
| -------- | -------- | ----- | ----------------------------- |
| `tokens` | `string` | ✅ Yes | 以逗号分隔的代币 mint 地址列表，用于获取其最新价格。 |

### 示例请求

```json
GET /api/v1/price-by-token?tokens=6Eh3zBrDPvGX3efeC4azjGy7zdKKrwGKuAwZQUzRpump,5heso9fVSTJ2vWwxBDuJUyKCeunTfvn5H5eV1kiJpump,H6EnL9x9ycxcNdQ8YnuxfE4iCnQFcKADC7eoGd9npump,DDEXoWMyDHpCNCyvg5Pakz5u9AmFzmBjjcs92Ux7pump,F9TgEJLLRUKDRF16HgjUCdJfJ5BK6ucyiW8uJxVPpump,So11111111111111111111111111111111111111112
```

### 响应示例

```json

[
    {
        "timestamp": "1737148622",
        "price_in_usd": 218.45145294,
        "token": "So11111111111111111111111111111111111111112",
        "decimals": 9,
        "name": "Wrapped SOL",
        "symbol": "SOL",
        "uri": "https://raw.githubusercontent.com/solana-labs/token-list/main/assets/mainnet/So11111111111111111111111111111111111111112/logo.png",
        "marketCap": 0,
        "supply": 0
    },
    {
        "timestamp": "1737148622",
        "token": "H6EnL9x9ycxcNdQ8YnuxfE4iCnQFcKADC7eoGd9npump",
        "price_in_sol": 0.000005035749216120721,
        "price_in_usd": 0.0011000667329030378,
        "decimals": 6,
        "name": "Telegram Engine",
        "symbol": "AIGRAM",
        "uri": "https://ipfs.io/ipfs/QmTwymvcXWrDoseVU9PfHmE79e4FCdXi84CGEQprjRwgUn",
        "marketCap": 1100042.67698022,
        "supply": 999978132.305888
    },
    ]


```

