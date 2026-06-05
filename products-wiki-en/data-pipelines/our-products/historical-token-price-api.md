---
description: Get latest price of SOL and SPL tokens
---

# Historical Token Price API

The Price API allows users to fetch historical token prices in both SOL and USD using indexed on-chain data.

**Base URL:**\
`https://graphql.astralane.io`

**Endpoint:**\
`GET /api/v1/dataset/price/history?token=<token_address>&from=<start_timestamp>&to=<end_timestamp>`

* **token:** The SPL token mint address (or "SOL" for native SOL).
* **from:** Start of the time range (Unix timestamp in seconds).
* **to:** End of the time range (Unix timestamp in seconds).

**Note:**\
Supports a maximum time range of 100 minutes per request.

**Description:**\
Returns historical price data for the specified token within the defined time window, with prices denominated in both SOL and USD. Ideal for charting, analytics, and time-series analysis.

### **Query params**

| Parameter | Type     | Required | Description                                                                                                                                  |
| --------- | -------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `token`   | `string` | ✅ Yes    | The `token` parameter accepts the mint address of the SPL token (or `"SOL"` for native Solana) whose historical prices you want to retrieve. |
| `from`    | integer  | ✅ Yes    | Unixtimestamp                                                                                                                                |
| `to`      | integer  |  ✅Yes    | Unixtimestamp                                                                                                                                |

### Sample request

```json
GET /api/v1/dataset/price/history?token=So11111111111111111111111111111111111111112&from=1739577600&to=1739577620
```

### Response

```json

[
    {
        "time": 1739577600,
        "value": 199.42621602747087
    }
]

```

