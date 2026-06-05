---
description: Raw solana transactions
---

# Trasaction Indexer

**Transaction Indexer Overview**

The **Transaction Indexer** captures and stores finalized Solana blockchain transactions from the latest two epochs using **Geyser gRPC**. This indexed data is made accessible via a **GraphQL API**, allowing developers and analysts to efficiently query and analyze transactions without directly interacting with Solana’s RPC endpoints.

***

#### **Key Features**

* **Real-Time Transaction Indexing**\
  Ingests and processes finalized transactions in real-time using Geyser gRPC.
* **Efficient Data Storage**\
  Transactions are structured and stored in **ClickHouse**, enabling high-performance querying.
* **GraphQL API Access**\
  Provides a robust and flexible GraphQL interface for querying transaction data, supporting filters and pagination.

***

#### **Architecture**

* **Data Ingestion**
  * **Source:** Solana blockchain via Geyser gRPC
  * **Processing:** Transactions are parsed, structured, and indexed
  * **Batching:** Batched and flushed periodically to optimize performance
* **Data Storage**
  * **Database:** ClickHouse (chosen for its speed and scalability)
* **GraphQL API**
  * Queryable endpoint for transaction data
  * Built-in resolvers support pagination, filtering, and deep transaction inspection

***

#### **GraphQL API**

* **Base URL:**\
  `https://graphql.astralane.io/api/v1/dataset/transactions`

***

#### **Query Example – Fetch Transactions with Pagination**

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

#### **Use Cases**

* Real-time monitoring and indexing of wallet activity
* Building custom transaction explorers
* Historical transaction analysis and token flow tracking
* Developer tools and dashboards for Solana on-chain data

***

### Query Filters

| Field              | Type    | Description                                             |
| ------------------ | ------- | ------------------------------------------------------- |
| `limit`            | Integer | Default: 100, max number of rows to fetch               |
| `page`             | Integer | Default: 1, Page number                                 |
| `block`            | Integer | Block slot to filter, eg: `307771651`                   |
| `start_block_time` | Integer | Unix timestamp of date time in seconds eg: `1734312900` |
| `end_block_time`   | Integer | Unix timestamp of date time in seconds`1734313900`      |
| `account_key`      | string  | Public key of account to search for                     |
| `start_block`      | Ineger  | Block slot to start with                                |
| `end_block`        | Integer | Block slot to end with                                  |
