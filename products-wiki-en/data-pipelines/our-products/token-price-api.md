---
description: Fetches the latest market price of SOL and supported SPL tokens.
---

# Token Price API

The Price API allows users to retrieve real-time token prices denominated in both SOL and USD, using the most up-to-date market data available.

**Base URL**

[**https://graphql.astralane.io**](https://graphql.astralane.io)

**Endpoint:**\
`GET /api/v1/price-by-token?tokens=token1,token2,token3`

**Description:**\
Retrieves the latest prices for the specified tokens (up to 50), using a comma-separated list of token mint addresses in the `tokens` query parameter. Prices are returned in both SOL and USD.

**Note:** A maximum of 50 token mint addresses can be queried per request.

### **Query params**

| Parameter | Type     | Required | Description                                                                          |
| --------- | -------- | -------- | ------------------------------------------------------------------------------------ |
| `tokens`  | `string` | ✅ Yes    | A comma-separated list of token mint addresses for which to fetch the latest prices. |

### Sample request

```json
GET /api/v1/price-by-token?tokens=6Eh3zBrDPvGX3efeC4azjGy7zdKKrwGKuAwZQUzRpump,5heso9fVSTJ2vWwxBDuJUyKCeunTfvn5H5eV1kiJpump,H6EnL9x9ycxcNdQ8YnuxfE4iCnQFcKADC7eoGd9npump,DDEXoWMyDHpCNCyvg5Pakz5u9AmFzmBjjcs92Ux7pump,F9TgEJLLRUKDRF16HgjUCdJfJ5BK6ucyiW8uJxVPpump,So11111111111111111111111111111111111111112
```

### Response

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

