---
description: Top profitable wallets by Pnl
---

# Profitable Wallets

The **Top Profitable Wallets API** delivers insights into the most successful wallets based on their trading activity. It aggregates key performance indicators, including:

* **Trade Volume**
* **Trade Count**
* **Win Rate**
* **Overall Profit and Loss (PnL)**

These metrics are calculated over a user-defined time period, offering a detailed view of wallet performance.

**Use Case:**\
Ideal for identifying high-performing traders, conducting competitive analysis, and understanding wallet-level behavior across the ecosystem.

#### **Query Parameters**

* **`time`** _(required)_:\
  Specifies the time window for calculating wallet metrics.\
  **Supported values:**
  * `2h` – 2 hours
  * `3h` – 3 hours
  * `5h` – 5 hours
  * `10h` – 10 hours
  * `24h` – 24 hours
  * `3d` – 3 days
  * `1W` – 1 week
* **`page`** _(optional)_:\
  Specifies the page number for paginated results.\
  **Default:** `1`
* **`limit`** _(optional)_:\
  Number of records to return per page.\
  **Default:** `100`

***

**Example Request:**

```
GET https://graphql.astralane.io/api/v1/dataset/profitable-wallets?time=24h&page=1&limit=50
```

This request fetches the top 50 most profitable wallets over the past 24 hours.

***

**Response Format**

The API returns a JSON array, where each object represents a wallet along with its performance metrics.

#### Sample Response

```json
[
  {
    "wallet": "HmD1XGGXLRnYvq537jmTUuhcRgecZF3NeNoFtnBv5ERX",
    "trade_volume": 15705.95985742,
    "trade_count": 149,
    "win_rate": 40,
    "pnl_overview": 478.16333295
  }
]
```

**Field Descriptions**

* **`wallet`**:\
  The unique identifier of the wallet (typically the public key or wallet address).
* **`trade_volume`**:\
  The total trade volume in USD executed by the wallet during the selected time period.
* **`trade_count`**:\
  The number of trades performed by the wallet within the specified time window.
* **`win_rate`**:\
  The percentage of trades that were profitable, calculated as the ratio of profitable trades to total trades.
* **`pnl_overview`**:\
  The net Profit and Loss (PnL) for the wallet over the selected time period.\
  A positive value indicates a profit, while a negative value indicates a loss.

***

**Use Cases**

* **Investor Analysis**:\
  Identify high-performing wallets that may be worth monitoring or mirroring for investment opportunities.
* **Market Research**:\
  Gain insights into the trading behavior and performance of top wallets across the ecosystem.
* **Competitive Intelligence**:\
  Compare wallet profitability to analyze trading strategies and uncover market trends.

***

**Pagination**

To handle large datasets, the API supports pagination using the `page` and `limit` query parameters:

* **`page`**: Specifies the page number of the result set.
* **`limit`**: Defines the number of records returned per page.\
  **Maximum supported limit per page:** 100.

