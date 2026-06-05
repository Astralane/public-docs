---
description: Historical token balance indexer
---

# Historical Token Portfolio

The **Historical Portfolio Index** is a purpose-built indexer that retrieves historical token balances for a specified wallet over a defined timeframe. It offers users a comprehensive view of their portfolio composition at various points in time, making it ideal for monitoring changes in portfolio value. The indexer supports balance queries at the end of each day, month, and year, as well as for custom time intervals, enabling flexible and detailed historical analysis.

### **Functionality**

**Flexible Timeframe-Based Balance Tracking**:

* **Flexible Timeframe-Based Balance Tracking:**\
  Retrieves historical token balances for a wallet across a user-defined date range. Enhanced support for End of Day (EOD), End of Month (EOM), and End of Year (EOY) snapshots enables precise and periodic balance assessments.
* **Historical Insights:**\
  Delivers token balance data at specific points within the selected timeframe, supporting in-depth analysis of portfolio performance, asset allocation, and historical value trends.

### **Methodology**

1. **Index Creation with Filters**:
   * The index is initialized with customizable filters that define:
     * **Account ID:** Specifies the target wallet address for historical analysis.
     * **Time Range:** Sets the desired timeframe using start and end timestamps, supporting balance snapshots at End of Day (EOD), End of Month (EOM), or End of Year (EOY).
   * **Backfilling Historical Data:**\
     Once the index is created, a backfill process is triggered to populate historical token balances by querying on-chain data. This process captures snapshots of token holdings over the specified period, accounting for all balance-affecting events such as transfers, trades, and staking activities.
   * **Data Retrieval:**\
     Users can access the aggregated historical balance data at any time via API endpoints. The data can be retrieved as cumulative totals or broken down day-by-day, and is organized by token—offering a granular view of portfolio composition and balance fluctuations over time.

**Technical Insights**

* **Data Collection:**\
  The indexer gathers data by parsing on-chain records and transaction histories within the defined timeframe, ensuring accurate and comprehensive tracking of wallet activity.
* **Time-Based Balance Tracking:**\
  Enables the generation of balance snapshots at the end of each day, month, or year within the selected period, providing structured and periodic portfolio insights.
* **Price Correlation:**\
  By capturing token balances over time, users can align this data with historical price feeds to analyze changes in portfolio value and assess performance trend

**Use Cases**

* **Historical Portfolio Analysis:**\
  Allows users to review past portfolio compositions and identify patterns or shifts in token holdings over time.
* **Performance Tracking:**\
  Facilitates the evaluation of historical performance for individual tokens or overall portfolio growth across specific time periods.
* **Tax and Compliance Reporting:**\
  Delivers precise historical balance records that support audits, tax filings, and adherence to regulatory requirements.

### **API Documentation**

**admin-base-url**:[https://indexer.astralane.io/](https://indexer.astralane.io/)

**index-dataset-url:** [https://graphql.astralane.io/](https://graphql.astralane.io/)

### **Authorization**

Each request must include an `x-api-key` header for authorization. Ensure that `<your_api_key>` is replaced with your actual API key.

**Index Creation Flow**

* **Create New Index**\
  `POST {admin-base}/index/account`\
  **Purpose:** Initializes a new historical portfolio index to begin tracking token balances for a specified wallet.
* **Start Backfill Job**\
  `POST {admin-base}/backfill/{index_id}`\
  **Purpose:** Launches a backfill job to fetch and populate historical balance data for the specified index ID.
* **Check Backfill Status**\
  `GET {admin-base}/backfill/{index_id}`\
  **Purpose:** Retrieves the current status of the backfill process, allowing users to monitor progress.
* **Fetch Historical Portfolio Data**\
  `POST {index-dataset-url}/api/v1/portfolio/{index_id}/graphql`\
  **Purpose:** Returns historical token balance data for the specified index ID across the selected time range.

**Index Creation Example**

1.  **Create New Index**\
    `POST {admin-base}/index/account`

    **Headers:**\
    `x-api-key: <your_api_key>`

    **Request Body:**\
    **Example Config 1:**\
    This configuration sets up the index to track the historical token balances of a specified wallet, generating end-of-day (EOD) balance snapshots for each day within the defined timeframe.

```json

{
  "name": "My Main Wallet Portfolio",
  "filters": [
    {
      "column": "account_id",
      "predicates": [
        {
          "type": "eq",
          "value": ["BQgboAHJKPnankxDw9GG8n9tnE2L6KA1mWiVLWnPAVma"]
        }
      ]
    },
    {
      "column": "time",
      "predicates": [
        {
          "type": "eq",
          "value": ["EOM"] // Choose from **EOM**(End of month), **EOY**(End of year)
        }
      ]
    }
  ]
}
```

**Example Config 2:**\
This configuration indexes the historical token balances of a specified wallet, generating daily end-of-day (EOD) balance snapshots over the defined time range.

```json
{
  "name": "My Main Wallet Portfolio",
  "filters": [
    {
      "column": "account_id",
      "predicates": [
        {
          "type": "eq",
          "value": ["BQgboAHJKPnankxDw9GG8n9tnE2L6KA1mWiVLWnPAVma"]
        }
      ]
    },
    {
      "column": "time",
      "predicates": [
         {
          "type": "gt",
          "value": ["1730246400"]
        },
        { 
          "type": "lt",
          "value": ["1731196800"]
        }
      ]
    }
  ]
}
```

**Description:** Initializes a new index to track historical portfolio data for a specified account.

**Sample Response**:

```json
{
    "message": "Account index was created",
    "id": "ea60d5d2-85d7-4bd2-82f3-c0d7fe934e33"
}
```

**2. Start Backfill Job**\
`POST {admin-base}/backfill/{index_id}`

**Headers:**\
`x-api-key: <your_api_key>`

**URL Parameters:**

* **index\_id:** The unique identifier of the index created in the previous step.

**Request Body**:

```json

{
  "action": "start"
}
```

**Description:** Initiates the backfill process to populate historical token balance data for the specified index within the defined timeframe.

**Sample Response**:

```json
{
    "action_type": "start",
    "index_id": "ea60d5d2-85d7-4bd2-82f3-c0d7fe934e33"
}
```

**3. Check Backfill Status**\
`GET {admin-base}/backfill/{index_id}`

**Headers:**\
`x-api-key: <your_api_key>`

**URL Parameters:**

* **index\_id:** The unique identifier of the index for which to check the backfill status.

**Description:**\
Fetches the current status of the backfill job, including progress updates and estimated time to completion.

**Sample Response**:

```json
json
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

**4. Fetch Historical Portfolio Data**\
`POST {index-dataset-url}/api/v1/portfolio/{index_id}/graphql`\
Or use the GraphQL Playground by pasting the following URL into your browser (replace `{index_id}` with your actual index ID):\
**GraphQL Playground URL:**\
`https://graphql.astralane.io/api/v1/portfolio/{index_id}/graphql`

**Headers:**\
`x-api-key: <your_api_key>`

**URL Parameters:**

* **index\_id:** The unique identifier of the index from which to retrieve historical portfolio data.

**Description:**\
Retrieves historical portfolio data for the specified index across the defined date range. This data includes token balances over time, enabling detailed analysis of portfolio changes.

**sample Query:**

```graphql
query BalanceData {
    balanceData {
        date
        total_balance_usd
        tokens {
            timestamp
            balance
            mint
            balance_usd
        }
    }
}

```

#### Example responses for end of the day token balances

```json
 {
        "data": {
            "balanceData": [
                {
                    "date": "2024-10-01",
                    "total_balance_usd": 97442.96595799158,
                    "tokens": [
                        {
                            "timestamp": "2024-10-01 00:00:00",
                            "balance": 912.255539,
                            "mint": "2b1kV6DkPAnxd5ixfnxCpjxmKwqjjaYmCZfHsFu24GXo",
                            "balance_usd": 912.1079505163582
                        },
                        {
                            "timestamp": "2024-10-01 00:00:00",
                            "balance": 350917.146019,
                            "mint": "bobaM3u8QmqZhY1HwAtnvze9DLXvkgKYk3td3t8MLva",
                            "balance_usd": 228.8904975123627
                        },
                  
                    ]
                },
                {
                    "date": "2024-10-02",
                    "total_balance_usd": 94190.14343633992,
                    "tokens": [
                        {
                            "timestamp": "2024-10-02 00:00:00",
                            "balance": 2611.5364,
                            "mint": "HZ1JovNiVvGrGNiiYvEozEVgZ58xaU3RKwX8eACQBCt3",
                            "balance_usd": 840.5278588025184
                        },
                        {
                            "timestamp": "2024-10-02 00:00:00",
                            "balance": 0.19602522900000002,
                            "mint": "jucy5XJ76pHVvtPZb5TKRcGQExkwit2P5s4vY8UzmpC",
                            "balance_usd": 29.091447467872708
                        }
                    ]
                },
                {
                    "date": "2024-10-03",
                    "total_balance_usd": 91889.01628388994,
                    "tokens": [
                        {
                            "timestamp": "2024-10-03 00:00:00",
                            "balance": 96000,
                            "mint": "7atgF8KQo4wJrD5ATGX7t1V2zVvykPJbFfNeVf1icFv1",
                            "balance_usd": 0.04259115659076336
                        },
                        {
                            "timestamp": "2024-10-03 00:00:00",
                            "balance": 2.579127254,
                            "mint": "MNDEFzGvMt87ueuHvVU9VcTqsAP5b3fTGPsHuuPA5ey",
                            "balance_usd": 0.27138132087748235
                        }
                    ]
                }
            ]
        }
    }
```

#### Example response for EOM (End of the month) token balances

```json
{
 "data": {
        "balanceData": [
           
            {
                "date": "2024-09-30",
                "total_balance_usd": 102344.14134378362,
                "tokens": [
                    {
                        "timestamp": "2024-09-30 00:00:00",
                        "balance": 0.37321068900000004,
                        "mint": "J1toso1uCk3RLmjorhTtrVwY9HJ7X8V9yYac6Y7kGCPn",
                        "balance_usd": 64.911234431185
                    },
                    {
                        "timestamp": "2024-09-30 00:00:00",
                        "balance": 4.602259,
                        "mint": "EKpQGSJtjMFqKZ9KQanSqYXRcF8fBopzLHYxdM65zcjm",
                        "balance_usd": 11.330136288092994
                    }
                ]
            },
            {
                "date": "2024-10-31",
                "total_balance_usd": 108291.15598369065,
                "tokens": [
                    {
                        **"timestamp": "2024-10-31 00:00:00",
                        "balance": 0.199743006,
                        "mint": "he1iusmfkpAdwvxLNGV8Y1iSbj4rUy6yMhEA3fotn9A",
                        "balance_usd": 35.163866912937515
                    },
                    {
                        "timestamp": "2024-10-31 00:00:00",
                        "balance": 296.22111298,
                        "mint": "hntyVP6YFm1Hg25TN9WGLqM12b8TQmcknKrdu1oxWux",
                        "balance_usd": 1868.362032881505
                    },
                ]
            },
            {
                "date": "2024-11-30",
                "total_balance_usd": 132234.13328501736,
                "tokens": [
                    {
                        "timestamp": "2024-11-30 00:00:00",
                        "balance": 4.602259,
                        "mint": "EKpQGSJtjMFqKZ9KQanSqYXRcF8fBopzLHYxdM65zcjm",
                        "balance_usd": 11.516714801692155
                    },
                    {
                        "timestamp": "2024-11-30 00:00:00",
                        "balance": 42168.796381,
                        "mint": "GQdF6BctM9JsoUqxjgMBpnzRDhvVEBPJK9SG7ykopump",
                        "balance_usd": 2.114111146960422
                    },
                    
                ]
            }
        ]
    }
}
```



```json
{
    "data": {
        "balanceData": [
            {
                "date": "2024-12-31",
                "total_balance_usd": 132317.44254361294,
                "tokens": [
                    {
                        "timestamp": "2024-12-31 00:00:00",
                        "balance": 200,
                        "mint": "3UfknvCm4No13GHBPwNvXqJt9kroZcPv3psWswqpzixt",
                        "balance_usd": 0.042376126026558644
                    },
                    {
                        "timestamp": "2024-12-31 00:00:00",
                        "balance": 0.0064,
                        "mint": "HZ1JovNiVvGrGNiiYvEozEVgZ58xaU3RKwX8eACQBCt3",
                        "balance_usd": 0.0027155486819310844
                    },
                    {
                        "timestamp": "2024-12-31 00:00:00",
                        "balance": 537.444440265,
                        "mint": "bSo13r4TkiE4KumL71LsHTPpL2euBYLFx6h9HP3piy1",
                        "balance_usd": 129025.68011636613
                    }
                ]
            }
        ]
    }
}
```



Demo : [https://www.loom.com/share/f9419d418c504eb6a048937ce25e227e](https://www.loom.com/share/f9419d418c504eb6a048937ce25e227e)
