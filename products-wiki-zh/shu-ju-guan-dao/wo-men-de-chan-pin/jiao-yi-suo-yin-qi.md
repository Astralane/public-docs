---
description: 原始 Solana 交易
---

# 交易索引器

**Transaction Indexer 概览**

该交易索引器通过 **Geyser gRPC** 捕获并存储最近两个 epoch 内已最终确认（finalized）的 Solana 区块链交易。索引后的数据通过 **GraphQL API** 提供给开发者与分析师使用，使其无需直接对接 Solana 的 RPC 端点即可高效查询与分析交易。

***

**关键特性（Key Features）**

* **实时交易索引（Real-Time Transaction Indexing）**：使用 Geyser gRPC 实时摄取并处理最终确认的交易。
* **高效数据存储（Efficient Data Storage）**：将交易结构化后存储于 ClickHouse，支持高性能查询。
* **GraphQL API 访问（GraphQL API Access）**：提供强大灵活的 GraphQL 查询接口，支持筛选与分页。

***

**架构（Architecture）**

* **数据摄取（Data Ingestion）**
  * **来源：**&#x901A;过 Geyser gRPC 从 Solana 区块链获取
  * **处理：**&#x5BF9;交易进行解析、结构化与索引
  * **批处理：**&#x6279;量写入并周期性 flush 以优化性能
* **数据存储（Data Storage）**
  * 数据库：ClickHouse（因其速度与可扩展性被选用）
* **GraphQL API**
  * 可查询的交易数据端点
  * 内置解析器支持分页、过滤以及对交易的深度检查

***

#### **GraphQL API**

* **基础地址（Base URL）：**\
  `https://graphql.astralane.io/api/v1/dataset/transactions`

***

**查询示例——带分页的交易查询**

```graphql
query Transaction {
  transaction(
    limit: 1, 
    page: 1, 
    start_block_time: 1734312900, 
    end_block_time: 1734323700, 
    account_key: "BRnAAgctCz19YZw6q7maRsXHXP1j4oeqC93PGzjE5Si8"
  ) {
    block
    blockTime
    blockHeight
    parentSlot
    previousBlockhash
    signature
    transaction {
      signatures
      message {
        accountKeys
        recentBlockhash
        header {
          numReadonlySignedAccounts
          numReadonlyUnsignedAccounts
          numRequiredSignatures
        }
        instructions {
          accounts
          data
          programIdIndex
        }
      }
      meta {
        fee
        preBalances
        postBalances
        logMessages
        innerInstructionsNone
        rewards
        loadedWritableAddresses
        loadedReadonlyAddresses
        returnDataNone
        computeUnitsConsumed
        innerInstructions {
          index
          instructions {
            accounts
            data
            programIdIndex
          }
        }
        preTokenBalances {
          accountIndex
          mint
          owner
          programId
          uiTokenAmount {
            uiAmount
            decimals
            amount
            uiAmountString
          }
        }
        postTokenBalances {
          accountIndex
          mint
          owner
          programId
          uiTokenAmount {
            uiAmount
            decimals
            amount
            uiAmountString
          }
        }
      }
    }
    blockHash
  }
}
```

***

**应用场景**

* 钱包活动的实时监控与索引
* 构建自定义交易浏览器
* 历史交易分析与代币流向追踪
* 面向 Solana 链上数据的开发者工具与看板

***

### 查询过滤项

| 字段                 | 类型      | 描述                                |
| ------------------ | ------- | --------------------------------- |
| `limit`            | Integer | 默认 100；最大拉取行数。                    |
| `page`             | Integer | 默认 1；页码。                          |
| `block`            | Integer | 按区块槽位过滤，例如`307771651`             |
| `start_block_time` | Integer | 开始时间的 Unix 时间戳（秒），例如 `1734312900` |
| `end_block_time`   | Integer | 结束时间的 Unix 时间戳（秒），例如 `1734313900` |
| `account_key`      | string  | 要检索的账户公钥。                         |
| `start_block`      | Ineger  | 起始区块槽位。                           |
| `end_block`        | Integer | 起始区块槽位。                           |
