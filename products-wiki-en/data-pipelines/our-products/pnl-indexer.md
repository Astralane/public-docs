---
description: Historical PnL Indexer
---

# PnL Indexer

The **Solana Account Indexer** is a specialized tool designed to accurately track and calculate the Profit and Loss (PnL) of any given Solana account across specified historical or current timeframes. Utilizing the First-In-First-Out (FIFO) accounting method, the indexer generates both cumulative and daily PnL metrics, empowering users to evaluate long-term gains or losses as well as day-to-day performance changes with precision

**Functionality**

* **Cumulative PnL:** Calculates the total Profit and Loss across all transactions for the specified time frame, offering a comprehensive view of overall account performance.
* **Daily PnL:** Provides a day-by-day breakdown of Profit and Loss, enabling users to assess daily performance and identify short-term trends in asset valuation.

**Methodology**

**FIFO (First-In-First-Out) Calculation:**

The FIFO methodology matches each outgoing asset (e.g., sale, trade) to the earliest-acquired position available, ensuring accurate and consistent cost-basis accounting. By systematically aligning asset transactions in chronological order, FIFO precisely reflects realized gains and losses, providing clarity even amid market volatility.

* **PnL Calculation**
  * **Realized PnL:** Determined by matching each outgoing asset (sold or transferred out) to the earliest-acquired position using the FIFO method, accurately capturing profits or losses upon asset disposal.
  * **Unrealized PnL:** Assessed by comparing the current market value of assets still held in the account against their original acquisition cost, reflecting potential gains or losses.
  * **Daily Tracking:** Monitors realized and unrealized PnL on a daily basis, enabling detailed visibility into each day's cumulative financial performance.

**Technical Insights**

* **Transaction Parsing:** The indexer systematically parses account transactions, accurately identifying activities such as deposits, withdrawals, and asset swaps to construct a comprehensive transaction history.
* **Price Updates:** To ensure precise daily PnL calculations, the indexer retrieves token prices at defined intervals (e.g., end-of-day), maintaining accurate valuations for unrealized gains or losses.
* **Daily Rollover:** The indexer performs a daily rollover, logging updated PnL data and retrieving the latest asset prices to maintain a continuous and reliable history for effective trend analysis.

**Use Cases**

* **Individual Account Analysis:** Enables users to gain comprehensive insights into their personal trading activities and performance.
* **Portfolio Management:** Assists portfolio managers in accurately monitoring, analyzing, and reporting daily Profit and Loss metrics across managed assets.
* **Tax Reporting:** Facilitates tax preparation by providing detailed historical data on realized gains and losses, simplifying compliance and reporting requirements.

### Try out the API on Swagger docs

Admin API - [https://indexer.astralane.io/api-docs#/](https://indexer.astralane.io/api-docs#/)

Index Data API - [https://graphql.astralane.io/api-docs](https://graphql.astralane.io/api-docs)

The indexer exposes a set of API endpoints designed to initialize indexing, backfill historical transaction data, and retrieve comprehensive cumulative and daily Profit and Loss (PnL) metrics for any specified account.

### Index creation flow

1.  **Create New Index**

    **POST** `{admin-base-url}/index/pnl`

    **Purpose**: Initializes a new index for tracking PnL data.
2.  **Start Backfill Job**

    **POST** `{admin-base-url}/index/{index_id}/backfill`

    **Purpose**: Starts a backfill job to populate historical PnL data based on the specified index ID.
3.  **Check Backfill Status**

    &#x20;**GET** `{admin-base-url}/index/{index_id}/backfill`

    **Purpose**: Check the current status of the backfill job for the specified index.
4.  **Fetch Cumulative PnL Data**

    &#x20;**GET** `{index-dataset-url}/api/v1/dataset/cumulative-pnl/{index_id}`

    **Purpose**: Retrieves the cumulative PnL data for the specified index ID over the specified time frame.
5.  **Fetch Daily PnL Data**

    &#x20;**GET** `{index-dataset-url}/api/v1/dataset/daily-pnl/{index_id}`

    **Purpose**: Retrieves the daily PnL data for the specified index ID over the specified time frame.

### Index creation example

**admin-base-url**:[https://indexer.astralane.io/](https://indexer.astralane.io/)

**index-dataset-url:** [https://graphql.astralane.io/](https://graphql.astralane.io/)

### 1. **Create a New Index**

**POST** `{admin-base}/index/pnl`

**Headers**:`x-api-key: <your_api_key>`

**Request Body**:

```json
{
    "name": "My Main Wallet Porfolio",
    "filters": [
        {
            "column": "account_id",
            "predicates": [
                {
                    "type": "eq",
                    "value": [
                        "AmTRLjX9T8KktBBnw1F78DyDb17fr6L8BNpEgCPGs5eM"
                    ]
                }
            ]
        },
        {
            "column": "time",
            "predicates": [
                {
                    "type": "gt",
                    "value": [
                        "1725148800"
                    ]
                },
                {
                    "type": "lt",
                    "value": [
                        "1730505600"
                    ]
                }
            ]
        }
    ]
}
```

**Description:** Initializes a new index to track Profit and Loss (PnL) data for the specified account over the provided date range.

**Sample Response:**

```json

{
    "message": "Index was created",
    "id": "ec5b37a2-c577-4e89-b5aa-077e10771419"
}
```

**Payload Details:**

**`name`**

* **Description:** A user-defined identifier for the index, enabling easy reference and retrieval in the future.
* **Example:** `"My Main Wallet Portfolio"`
* **Purpose:** Helps clearly label and organize indexes according to specific accounts, portfolios, or tracking purposes.

**`filters`**

* **Description:** An array of filter objects, each containing criteria to specify which transactions should be included in the index.
* **Purpose:** Filters establish specific conditions to determine which transactions and data are included within the index, with each filter targeting particular `columns` using defined `predicates` for precise data selection.
* **Components:** Each filter consists of a `column` field and a corresponding array of `predicates`.

#### **Filter 1: Account ID Filter**

* **`column`: `"account_id"`**:
  * **Description:** Filters transactions by the `account_id` column, ensuring the index includes only transactions associated with the specified Solana account.
  * **`predicates`**:
    * **Type:** `"eq"` (equals) — Matches transactions exactly to the specified `account_id`
    * **Value**: An array containing the user's specific Solana **account ID**.
    * **User-Specific Element**: The account ID uniquely identifies the user's Solana wallet address.
    * **Example Value**: `"4BjxyW2GTD1qBwUk96mAR7dV1A3Pq51hPx78WLqrXegX"`
    * **Purpose**: Limits the index to include data solely related to the user's designated account.

#### **Filter 2: Time Filter**

* **`column`: `"time"`**:
  * **Description**: This filter applies to the `time` column, enabling users to specify an exact date and time range for transaction inclusion.
  * **`predicates`**:
    * The `predicates` array includes two conditions:
      1. **Greater Than (`gt`)**:
         * **Type**: `"gt"` — Indicates that the time must be greater than the specified value.
         * **Description**: The initial time of a range, given as a Unix timestamp (in seconds).
         * **User-Created Timestamp**: User-defined start datetime
         * **Example Value**: `"1718148102"`
      2. **Less Than (`lt`)**:
         * **Type**: `"lt"` — Indicates that the time must be earlier than the specified value.
         * **Description**: The end time of the range, given as a Unix timestamp&#x20;
         * **User-Specific Element**: The end timestamp is user-defined, representing the upper limit of the desired time range.
         * **Example Value**: `"1718339227"`
    * **Purpose**: To include only transactions that fall within the user-defined time range. This timeframe is fully customizable, allowing tailored analysis based on specific periods.

### 2. **Start Backfill Job**

**POST** `{admin-base-url}/index/{index_id}/backfill`

**Headers**:`x-api-key`: `<your_api_key>`

**URL Parameters**:

`index_id`: The ID of the index created in the previous step.

**Request Body**:

```json
{
  "action": "start"
}
```

**Description**: Initiates the backfill process for the specified index ID, retrieving and processing historical data to populate corresponding Profit and Loss (PnL) records.

**Sample Response**:

```json
{
  "action_type": "start",
  "index_id": "ec5b37a2-c577-4e89-b5aa-077e10771419"
}
```

### 3. **Check Backfill Status**

**GET** `{admin-base-url}/index/{index_id}/backfill`

**Headers**:`x-api-key`: `<your_api_key>`

**URL Parameters**:

`index_id`: The ID of the index to check the backfill status.

**Description**: Retrieves the current status of the backfill job, enabling users to track its progress and monitor completion.

**Sample Response**:

```json
{
    "index_id": "a82a9871-54ee-4adb-9bd6-5dfcd418b2ae",
    "start_time": "2024-10-26 20:36:28.000",
    "update_time": "2024-10-26 20:37:03.000",
    "status": "completed",
    "percentage": 1,
    "last_processed_block": 7,
    "starting_block": 0,
    "error_code": 0,
    "error_message": ""
}
```

### **4. Fetch Cumulative PnL Data**

**GET** `{index-dataset-url}/api/v1/dataset/cumulative-pnl/{index_id}`

**Headers**:`x-api-key`: `<your_api_key>`

**URL Parameters**:

`index_id`: The unique identifier(ID) of the index for which cumulative PnL data is to be retrieved.

**Description**: Returns the cumulative Profit and Loss (PnL) data for a specified index within the given date range, providing a comprehensive overview of overall performance.

**Sample Response:**

```json
{
    "slot_start": "297410893",
    "slot_end": "297518104",
    "time_from": "2024-10-24 00:00:00",
    "time_to": "2024-10-26 23:23:23",
    "realized_pnl_usd": 193167.09600963892,
    "unrealized_pnl_usd": 671366.3884107296,
    "tokens_traded": [
        {
            "timestamp": "2024-10-24 16:11:04",
            "block_slot": "297450632",
            "transaction_id": "qSBkprVyGvKeA64rnNycFWDhQEvzt84FKHYC97JREgQwEBivF543zS78zxDaNipSyjEx2n6tRLZnk8KDkQKvdwy",
            "mint": "So11111111111111111111111111111111111111112",
            "shares": 79.18219076,
            "trade_type": "BUY",
            "dex_name": "   ",
            "price_in_usd": 176.5638905707737,
            "token_running_balance": 13980.715664502766
        },
        {
            "timestamp": "2024-10-24 21:37:06",
            "block_slot": "297493708",
            "transaction_id": "45KVtqCfwUBDDRFRGLqnypu2qdCaVpYQp2Pa6vFgnYrnbNA85Ye3KDxWoJFRe6a25nmvHcYn47wLjx52ZFoGB3P7",
            "mint": "Bz4MhmVRQENiCou7ZpJ575wpjNFjBjVBSiVhuNg1pump",
            "shares": 3683402.913021,
            "trade_type": "SELL",
            "dex_name": "   ",
            "price_in_usd": 0.017859178498502083,
            "token_running_balance": 65782.55010554458
        },
        {
            "timestamp": "2024-10-24 21:37:06",
            "block_slot": "297493708",
            "transaction_id": "45KVtqCfwUBDDRFRGLqnypu2qdCaVpYQp2Pa6vFgnYrnbNA85Ye3KDxWoJFRe6a25nmvHcYn47wLjx52ZFoGB3P7",
            "mint": "So11111111111111111111111111111111111111112",
            "shares": 293.220454318,
            "trade_type": "BUY",
            "dex_name": "   ",
            "price_in_usd": 177.8712714605693,
            "token_running_balance": 52155.495027788435
        },
        {
            "timestamp": "2024-10-24 21:38:54",
            "block_slot": "297493954",
            "transaction_id": "5Y2LXVRMby1HmjmUoMbKtCLicYGPgagZDwv1sMFLw7FH2K56Y4UBWN43E8nQAPKhV8dd9KwN6vm3RyPiPgcRNL56",
            "mint": "Bz4MhmVRQENiCou7ZpJ575wpjNFjBjVBSiVhuNg1pump",
            "shares": 3315062.621719,
            "trade_type": "SELL",
            "dex_name": "   ",
            "price_in_usd": 0.017859178498502083,
            "token_running_balance": 59204.29509499189
        },
        {
            "timestamp": "2024-10-24 21:38:54",
            "block_slot": "297493954",
            "transaction_id": "5Y2LXVRMby1HmjmUoMbKtCLicYGPgagZDwv1sMFLw7FH2K56Y4UBWN43E8nQAPKhV8dd9KwN6vm3RyPiPgcRNL56",
            "mint": "So11111111111111111111111111111111111111112",
            "shares": 229.160980905,
            "trade_type": "BUY",
            "dex_name": "   ",
            "price_in_usd": 177.8712714605693,
            "token_running_balance": 40761.155042723585
        }
      
    ],
    "tokens_not_traded": [
        {
            "timestamp": "2024-10-24 11:11:05",
            "block_slot": "297410893",
            "transaction_id": "3ndGPr6rPVCmakD5BjECCcknXggWMmbV4b4JaGvxMBxSaKeMdJpFQwQyiCKt9vBww8BDLkyaRk3yf5faxK3tbq7m",
            "mint": "So11111111111111111111111111111111111111112",
            "shares": 4,
            "trade_type": "SELL",
            "dex_name": "   ",
            "price_in_usd": 172.2682353857461,
            "token_running_balance": 689.0729415429844
        },
        {
            "timestamp": "2024-10-24 14:31:21",
            "block_slot": "297437409",
            "transaction_id": "4mQbHRRdDhFDkyTiFaTYets7ZNYLgCM53BmopS5hQvxeSLUZr3ZbcG1oxA2U62fXURbp4eiwCjNUdrmABd7SzTYc",
            "mint": "So11111111111111111111111111111111111111112",
            "shares": 6,
            "trade_type": "SELL",
            "dex_name": "   ",
            "price_in_usd": 175.5622513596877,
            "token_running_balance": 1053.373508158126
        }
    ]
}
```

### **5. Fetch Daily PnL Data**

**GET** `{index-dataset-url}/api/v1/dataset/daily-pnl/{index_id}`

**Headers**:`x-api-key`: `<your_api_key>`

**URL Parameters**:

`index_id`: The ID of the index to fetch daily PnL data.

**Description**: Fetches the daily PnL data for the specified index within the provided date range.

**URL Parameters:**

* **index\_id:** The unique identifier of the index for which daily Profit and Loss (PnL) data is to be retrieved.
* **Description:** Returns daily PnL data for the specified index across the defined date range, offering detailed insights into day-by-day performance.

**Sample Response**:

```json
[
    {
        "date": "2024-10-24",
        "realized_pnl": -1594473.0642288465,
        "unrealized_pnl": -3018.3074229327203,
        "tokens_traded": [
            
            {
                "timestamp": "2024-10-24 14:31:21",
                "block_slot": "297437409",
                "transaction_id": "4mQbHRRdDhFDkyTiFaTYets7ZNYLgCM53BmopS5hQvxeSLUZr3ZbcG1oxA2U62fXURbp4eiwCjNUdrmABd7SzTYc",
                "mint": "Bz4MhmVRQENiCou7ZpJ575wpjNFjBjVBSiVhuNg1pump",
                "shares": 45395648.422744,
                "trade_type": "BUY",
                "dex_name": "   ",
                "price_in_usd": 0.0005947846472226818,
                "token_running_balance": 27000.63473256668
            },
            {
                "timestamp": "2024-10-24 14:31:21",
                "block_slot": "297437409",
                "transaction_id": "4mQbHRRdDhFDkyTiFaTYets7ZNYLgCM53BmopS5hQvxeSLUZr3ZbcG1oxA2U62fXURbp4eiwCjNUdrmABd7SzTYc",
                "mint": "So11111111111111111111111111111111111111112",
                "shares": 6,
                "trade_type": "SELL",
                "dex_name": "   ",
                "price_in_usd": 175.5622513596877,
                "token_running_balance": 1053.373508158126
            },
            {
                "timestamp": "2024-10-24 16:11:04",
                "block_slot": "297450632",
                "transaction_id": "qSBkprVyGvKeA64rnNycFWDhQEvzt84FKHYC97JREgQwEBivF543zS78zxDaNipSyjEx2n6tRLZnk8KDkQKvdwy",
                "mint": "Bz4MhmVRQENiCou7ZpJ575wpjNFjBjVBSiVhuNg1pump",
                "shares": 8561619.292529,
                "trade_type": "SELL",
                "dex_name": "   ",
                "price_in_usd": 0.0009076950968016275,
                "token_running_balance": 7771.339852510792
            },
            {
                "timestamp": "2024-10-24 16:11:04",
                "block_slot": "297450632",
                "transaction_id": "qSBkprVyGvKeA64rnNycFWDhQEvzt84FKHYC97JREgQwEBivF543zS78zxDaNipSyjEx2n6tRLZnk8KDkQKvdwy",
                "mint": "So11111111111111111111111111111111111111112",
                "shares": 79.18219076,
                "trade_type": "BUY",
                "dex_name": "   ",
                "price_in_usd": 176.5638905707737,
                "token_running_balance": 13980.715664502766
            }
        ],
        "tokens_cost_basis": []
    },
    {
        "date": "2024-10-25",
        "realized_pnl": -1764720.7775157518,
        "unrealized_pnl": 0,
        "tokens_traded": [
            {
                "timestamp": "2024-10-25 00:36:13",
                "block_slot": "297517893",
                "transaction_id": "4v1wHSS92iahBveM8FhNXRhbkMvfEdWQ45o173MDJjkpNAbAZmMZeEkcgKoBp5raxkowtFirpCx26MBv2az4BiYU",
                "mint": "Bz4MhmVRQENiCou7ZpJ575wpjNFjBjVBSiVhuNg1pump",
                "shares": 2983556.359547,
                "trade_type": "SELL",
                "dex_name": "   ",
                "price_in_usd": 0.014919479377863682,
                "token_running_balance": 44513.1075789555
            },
            {
                "timestamp": "2024-10-25 00:36:13",
                "block_slot": "297517893",
                "transaction_id": "4v1wHSS92iahBveM8FhNXRhbkMvfEdWQ45o173MDJjkpNAbAZmMZeEkcgKoBp5raxkowtFirpCx26MBv2az4BiYU",
                "mint": "So11111111111111111111111111111111111111112",
                "shares": 241.968258132,
                "trade_type": "BUY",
                "dex_name": "   ",
                "price_in_usd": 176.6293329283558,
                "token_running_balance": 42738.69202369136
            }
        ],
        "tokens_cost_basis": []
    }
]
```
