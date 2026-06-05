---
description: The API provides key metrics to evaluate the trading performance of wallets
---

# Wallet PnL API

The **Wallet PnL API** delivers detailed Profit and Loss (PnL) insights for a specific wallet over a user-defined time period. It aggregates essential trading metrics, including:

* **Trade Volume**
* **Trade Count**
* **Win Rate**
* **Overall PnL**

**Use Case:**\
Ideal for tracking and analyzing the trading performance of individual wallets—whether for personal portfolio monitoring, research, or competitive benchmarking.

**Endpoint:**\
`GET /api/v1/dataset/wallet-pnl`

**Description:**\
Returns detailed Profit and Loss (PnL) metrics for a specified wallet over a selected time window. The response includes trade volume, trade count, win rate, and overall PnL, providing a comprehensive view of the wallet's trading performance.

***

#### **Query Parameters**

* **`time`** _(required)_:\
  The time window for calculating PnL metrics.\
  **Supported values:**
  * `2h` – 2 hours
  * `3h` – 3 hours
  * `5h` – 5 hours
  * `10h` – 10 hours
  * `24h` – 24 hours
* **`wallet`** _(required)_:\
  The wallet address (public key) for which to retrieve PnL data.

***

#### **Example Request**

```
GET /api/v1/dataset/wallet-pnl?time=24h&wallet=F18ibDvGaj4Yjzk1UfWCk5jE9zoy1JYkXNTCBwPuZpyf HTTP/1.1
Authorization: Bearer YOUR_API_KEY
```

This request fetches the wallet’s PnL metrics for the last 24 hours.

**Response Format**

The API returns a JSON object containing key performance metrics for the specified wallet.

***

#### **Sample Response**

```json
{
  "wallet": "F18ibDvGaj4Yjzk1UfWCk5jE9zoy1JYkXNTCBwPuZpyf",
  "trade_volume": 12500.75,
  "trade_count": 98,
  "win_rate": 45,
  "pnl_overview": 650.32
}
```

***

#### **Field Descriptions**

* **`wallet`**:\
  The unique wallet address provided in the request.
* **`trade_volume`**:\
  Total trade volume (in USD) executed by the wallet during the specified time window.
* **`trade_count`**:\
  Number of trades performed by the wallet in the given timeframe.
* **`win_rate`**:\
  Percentage of trades that resulted in profit.\
  &#xNAN;_&#x43;alculated as:_\
  `(Number of Winning Trades / Total Trades with Non-Zero Sell Amount) × 100`
* **`pnl_overview`**:\
  Net Profit and Loss (PnL) for the wallet during the selected period.\
  &#xNAN;_&#x50;ositive value = Profit, Negative value = Loss_

***

#### **Use Cases**

* **Personal Performance Monitoring**:\
  Evaluate your wallet’s trading performance over time to identify strengths and areas for improvement.
* **Investor Due Diligence**:\
  Assess the historical profitability of a wallet before making investment decisions, forming partnerships, or mirroring trading activity.
* **Trading Strategy Analysis**:\
  Analyze wallet-level PnL data to refine or validate trading strategies based on actual performance.
