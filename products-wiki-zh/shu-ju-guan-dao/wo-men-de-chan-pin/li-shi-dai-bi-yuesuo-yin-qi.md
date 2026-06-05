---
description: 历史代币余额索引器（Historical token balance indexer）
---

# 历史代币余额索引器

**Historical Portfolio Index** 是一款面向钱包级别的历史余额索引器，可在指定时间范围内检索某个钱包的历史代币余额。在多个时间点给出资产构成与价值变化，支持按日收盘（EOD）、月末（EOM）、年末（EOY）以及自定义时间区间的余额快照，便于开展灵活、细致的历史分析。

### 功能特性

* **基于时间窗口的灵活余额跟踪（Flexible Timeframe-Based Balance Tracking）：**\
  在用户定义的日期范围内检索钱包的历史代币余额，支持 EOD/EOM/EOY 快照，便于周期性与精确评估。
* **历史洞察（Historical Insights）：**\
  在所选时间窗内返回特定时间点的余额数据，支持对资产组合表现、资产配置与历史价值趋势进行深度分析。

### 方法论

1. **通过过滤器创建索引（Index Creation with Filters）**
   * 索引在初始化时可配置自定义过滤条件，用于限定数据范围：
     * **Account ID：**&#x6307;定进行历史分析的目标钱包地址。
     * **Time Range：**&#x901A;过起止时间戳设定分析时间窗，支持按日收盘（EOD）、月末（EOM）、年末（EOY）的余额快照。
   * **Backfilling Historical Data:**\
     索引创建后会触发回填流程，通过查询链上数据填充历史代币余额。该流程在指定期间内抓取持仓快照，并将转账、交易、质押等所有影响余额的事件纳入计算。
   * **Data Retrieval:**\
     用户可随时通过 API 端点获取聚合后的历史余额数据。数据既可按累计总额返回，也可按日拆分，并按代币组织，提供资产构成与余额波动的细粒度视图。

**技术要点（Technical Insights）**

* **数据采集（Data Collection）：**&#x5728;设定时间窗内解析链上记录与交易历史，确保对钱包活动的准确、全面跟踪。
* **基于时间的余额追踪（Time-Based Balance Tracking）：**&#x652F;持在所选区间内生成每日、每月、每年末的余额快照，形成结构化、周期性的组合洞察。
* **价格关联（Price Correlation）：**&#x901A;过连续记录代币余额，用户可结合历史价格数据分析组合价值变化并评估绩效趋势。

**应用场景（Use Cases）**

* **历史资产组合分析（Historical Portfolio Analysis）：**&#x56DE;顾历史资产构成，识别持仓变化模式或结构迁移。
* **绩效跟踪（Performance Tracking）：**&#x8BC4;估特定区间内单币种或整体组合的历史表现与增长。
* **税务与合规报告（Tax and Compliance Reporting）：**&#x63D0;供精确的历史余额记录，支持审计、纳税申报与合规要求。

### API 文档（API Documentation）

**admin-base-url**:[https://indexer.astralane.io/](https://indexer.astralane.io/)

**index-dataset-url:** [https://graphql.astralane.io/](https://graphql.astralane.io/)

### 授权（Authorization）

每个请求都必须在请求头中包含 `x-api-key` 用于鉴权。请将 `<your_api_key>` 替换为你的实际 API Key。

**索引创建流程（Index Creation Flow）**

1. **创建新索引（Create New Index）**\
   `POST {admin-base}/index/account`\
   **用途：**&#x4E3A;指定钱包初始化历史资产组合索引，用于开始跟踪代币余额。
2. **启动回填任务（Start Backfill Job）**\
   `POST {admin-base}/backfill/{index_id}`\
   **用途：**&#x4E3A;指定 index\_id 启动回填任务，抓取并填充历史余额数据。
3. **查询回填状态（Check Backfill Status）**\
   `GET {admin-base}/backfill/{index_id}`\
   **用途：**&#x83B7;取回填进度与当前状态，便于监控任务执行。
4. **获取历史资产组合数据（Fetch Historical Portfolio Data）**\
   `POST {index-dataset-url}/api/v1/portfolio/{index_id}/graphql`\
   **用途：**&#x5728;所选时间范围内返回指定 index\_id 的历史代币余额数据。
5.  **Create New Index**\
    `POST {admin-base}/index/account`\
    **Headers:**

    `x-api-key: <your_api_key>`

    **请求体（Request Body）**\
    **示例配置 1（Example Config 1）：**\
    该配置为指定钱包建立索引，并在定义的时间范围内为每一天生成日终（EOD）余额快照。

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

**示例配置 2（Example Config 2）**\
该配置为指定钱包建立索引，在给定时间范围内按天生成日终（EOD）余额快照。

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

**说明:** 为指定账户初始化一个用于跟踪历史资产组合数据的新索引。

**示例响应：**

```json
{
    "message": "Account index was created",
    "id": "ea60d5d2-85d7-4bd2-82f3-c0d7fe934e33"
}
```

**2. 启动回填任务（Start Backfill Job**）\
`POST {admin-base}/backfill/{index_id}`

**Headers:**\
`x-api-key: <your_api_key>`

**URL 参数：**

* **index\_id:** 上一步创建的索引唯一标识。

**请求体（Request Body）：**

```json

{
  "action": "start"
}
```

**说明：**&#x4E3A;指定的 index\_id 启动回填流程，在定义的时间范围内填充历史代币余额数据。<br>

**示例响应：**

```json
{
    "action_type": "start",
    "index_id": "ea60d5d2-85d7-4bd2-82f3-c0d7fe934e33"
}
```

**3. 查询回填状态（Check Backfill Status）**\
`GET {admin-base}/backfill/{index_id}`

**Headers:**\
`x-api-key: <your_api_key>`

**URL 参数：**

* **index\_id:** 需要查询回填状态的索引唯一标识。

**说明**：获取回填任务的当前状态，包括进度更新与预计完成时间。<br>

**示例响应：**

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

**4. 获取历史资产组合数据（Fetch Historical Portfolio Data）**\
`POST {index-dataset-url}/api/v1/portfolio/{index_id}/graphql`\
或在浏览器中使用 GraphQL Playground（将 `{index_id}` 替换为实际索引 ID）：<br>

**GraphQL Playground URL:**\
`https://graphql.astralane.io/api/v1/portfolio/{index_id}/graphql`

**Headers:**\
`x-api-key: <your_api_key>`

**URL 参数：**

* **index\_id:** 要检索历史资产组合数据的索引唯一标识。

**说明：**\
获取指定 index 在定义日期范围内的历史资产组合数据。返回的内容包含随时间变化的各代币余额，便于对资产组合变化进行细致分析。

**示例查询**（sample Query）：

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

**按日收盘（EOD）代币余额的示例响应**

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

**月末（EOM）代币余额的示例响应**

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
