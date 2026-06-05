---
description: 历史盈亏（PnL）索引器
---

# PnL 索引器

**Solana Account Indexer** 是一款专用工具，可在指定的历史或当前时间范围内，精准追踪并计算任意 Solana 账户的盈亏（PnL）。索引器采用先进先出（FIFO）会计方法，生成累计与按日维度的 PnL 指标，帮助用户同时把握长期收益/亏损与日常表现变化。

### **功能特性**

* **累计 PnL（Cumulative PnL）**：在指定时间范围内，汇总该账户所有交易的总盈亏，全景展示整体账户表现。
* **每日 PnL（Daily PnL）**：提供逐日盈亏明细，便于评估日度表现并识别短期趋势。

### **方法论**

**FIFO（先进先出）计算：**\
将每笔资产流出（如卖出、兑换）与最早获取的持仓一一匹配，确保成本计算一致与准确。通过将交易按时间顺序系统化匹配，FIFO 能精确反映已实现盈亏，在波动行情下亦能保持清晰可解释。

**PnL 计算口径**

* **已实现盈亏（Realized PnL）**：基于 FIFO，将每笔卖出/转出与最早购入的持仓匹配，准确记录处置时点的盈亏。
* **未实现盈亏（Unrealized PnL）**：将当前持有资产的市值与其历史成本对比，反映潜在收益或亏损。
* **日度跟踪（Daily Tracking）**：按日监控已实现与未实现盈亏，为当日累计财务表现提供细粒度可视化。

**技术要点（Technical Insights）**

* **交易解析（Transaction Parsing）：**&#x7CFB;统化解析账户交易，准确识别充值、提现、资产兑换等活动，构建完整的交易历史。
* **价格更新（Price Updates）：**&#x4E3A;确保日度 PnL 精度，按设定时间点（如收盘/日终）获取代币价格，用于未实现盈亏估值。
* **日切（Daily Rollover）：**&#x6BCF;日滚动更新，记录新的 PnL 数据并拉取最新价格，形成连续、可靠的历史序列，便于趋势分析。

**典型场景（Use Cases）**

* **个人账户分析：**&#x5168;面洞察个人交易活动与绩效。
* **资产组合管理：**&#x5E2E;助投资经理精准监控、分析并出具每日盈亏报表。
* **税务申报：**&#x63D0;供详细的历史已实现盈亏数据，简化合规申报与报告。

### 在线试用（Swagger 文档）

* 管理端 Admin API 文档：[https://indexer.astralane.io/api-docs#/](https://indexer.astralane.io/api-docs#/)
* 数据查询 Index Data API 文档：[https://graphql.astralane.io/api-docs](https://graphql.astralane.io/api-docs)

该索引器提供一组 API，用于初始化索引、回填历史交易数据，并获取任意账户的累计与每日 PnL 指标。

### 索引创建流程（Index creation flow）

1. **创建新索引（Create New Index）**\
   **POST** `{admin-base-url}/index/pnl`\
   **用途：**&#x521D;始化一个新的 PnL 跟踪索引。
2. **启动回填任务（Start Backfill Job）**\
   **POST** `{admin-base-url}/index/{index_id}/backfill`\
   **用途：**&#x6839;据指定 index\_id 启动历史数据回填，补齐对应的 PnL 数据。
3. **查询回填状态（Check Backfill Status）**\
   **GET** `{admin-base-url}/index/{index_id}/backfill`\
   **用途：**&#x67E5;看该 index 的回填进度与当前状态。
4. **获取累计 PnL（Fetch Cumulative PnL Data）**\
   **GET** `{index-dataset-url}/api/v1/dataset/cumulative-pnl/{index_id}`\
   **用途：**&#x6309;指定时间范围，返回该 index 的累计盈亏数据。
5. **获取每日 PnL（Fetch Daily PnL Data）**\
   **GET** `{index-dataset-url}/api/v1/dataset/daily-pnl/{index_id}`\
   **用途：**&#x6309;指定时间范围，返回该 index 的逐日盈亏数据。

### 索引创建示例（Index creation example）

**admin-base-url:** [https://indexer.astralane.io/](https://indexer.astralane.io/)

**index-dataset-url:** [https://graphql.astralane.io/](https://graphql.astralane.io/)

### 1. 创建新索引（Create a New Index）

**POST** `{admin-base}/index/pnl`

**Headers**:`x-api-key: <your_api_key>`

**查询示例:**

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

**说明：**&#x4E3A;指定账户与日期范围初始化一个新的 PnL 索引。

**示例响应：**

```json

{
    "message": "Index was created",
    "id": "ec5b37a2-c577-4e89-b5aa-077e10771419"
}
```

**请求载荷说明（Payload Details）**

**`name`**

* **描述：**&#x7D22;引的人性化名称，便于后续检索与引用。
* **示例：**"My Main Wallet Portfolio"
* **目的：**&#x6309;账户/组合/用途清晰标注并组织索引。

**`filters`**

* **描述：**&#x8FC7;滤条件数组，定义哪些交易将被纳入索引。
* **目的：**&#x901A;过指定`列`与`谓词`，实现精确的数据筛选。
* **组成：**&#x6BCF;个过滤器包含 `column` 字段与 `predicates` 数组。



**过滤器 1：账户过滤（Account ID Filter）**

* **`column`**: **`"account_id"`**
  * **描述**：按 `account_id` 列过滤，仅纳入该账户的交易。
* **`predicates`**：
  * **类型**：`"eq"`（等于）— 精确匹配指定的 `account_id`
  * **值**：包含用户 Solana **账户地址**的数组
  * **用户要素**：账户地址唯一标识用户的钱包
  * **示例值**：`"4BjxyW2GTD1qBwUk96mAR7dV1A3Pq51hPx78WLqrXegX"`
  * **目的**：仅纳入目标账户相关的数据。



**过滤器 2：时间过滤（Time Filter）**

* **`column`**: `"`**`time`**`"`
  * **描述**：按时间列过滤，限定交易的起止时间范围。
  * **`predicates`**：包含两个条件
  * **1 - 大于（`gt`）：**
    * **类型**：`"gt"`
    * **描述**：区间起点，Unix 时间戳（秒）
    * **用户自定义时间戳：**&#x7528;户定义的起始日期时间。
    * **示例**：`"1718148102"`
  * **2 - 小于（I`t`）：**
    * **类型**：`"lt"`
    * **描述**：区间终点，Unix 时间戳（秒）
    * **用户自定义元素：**&#x7ED3;束时间戳由用户指定，代表所需时间范围的上限。
    * **示例**："1718339227"
* **目的**：仅纳入落在用户自定义时间范围内的交易。该时间窗口完全可配置，便于围绕特定时期开展定制化分析。

### 2. 启动回填任务

**POST** `{admin-base-url}/index/{index_id}/backfill`

**Headers**:`x-api-key`: `<your_api_key>`

**URL Parameters**:

`index_id`: 上一步创建索引返回的 ID

**查询示例:**

```json
{
  "action": "start"
}
```

**说明：**&#x4E3A;指定 index\_id 启动回填，抓取与处理历史数据以生成对应的 PnL 记录。

**Sample Response**:

```json
{
  "action_type": "start",
  "index_id": "ec5b37a2-c577-4e89-b5aa-077e10771419"
}
```

### 3. 查询回填状态

**GET** `{admin-base-url}/index/{index_id}/backfill`

**Headers**:`x-api-key`: `<your_api_key>`

**URL Parameters**:

`index_id`: 要查询回填状态的索引 ID

**说明：**&#x8FD4;回回填任务的当前状态与进度，便于跟踪直至完成。

**示例响应：**

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

### **4.** 获取累计 PnL

**GET** `{index-dataset-url}/api/v1/dataset/cumulative-pnl/{index_id}`

**Headers**:`x-api-key`: `<your_api_key>`

**URL 参数：**

`index_id`: 要查询的索引唯一标识

**说明：**&#x8FD4;回指定索引在给定日期范围内的累计盈亏概览。

**示例响应：**

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

### **5.** 获取日度 PnL

**GET** `{index-dataset-url}/api/v1/dataset/daily-pnl/{index_id}`

**Headers**:`x-api-key`: `<your_api_key>`

**URL参数：**

`index_id`: 要拉取日度 PnL 的索引 ID（唯一标识）

**URL Parameters:**

* **index\_id:** 要拉取日度 PnL 的索引 ID（唯一标识）
* **Description:** 返回指定索引在定义日期范围内的日度 PnL 数据，提供逐日细化的绩效洞察。

**示例响应：**

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
