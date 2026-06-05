---
description: API for dex pools latest balances
---

# Dex Pools API

The **Dex Pools API** provides a comprehensive interface to query and retrieve detailed information about all supported Decentralized Exchange (DEX) pools. This API delivers real-time data on pool balances, including the latest token balances, enabling developers to access up-to-date liquidity information for seamless integration into applications.

Designed with performance and scalability in mind, the Dex Pools API ensures fast and reliable access to critical pool data, making it an essential tool for building decentralized finance (DeFi) applications, analytics platforms, and trading tools

**Base URL:**

[https://graphql.astralane.io/api/v1/dataset/dex-pools/v2](https://graphql.astralane.io/api/v1/dataset/dex-pools/v2)

### GraphQL Query

```graphql
query PoolsData {
  poolsData(page: 1) {
    block_slot
    block_time
    pool
    mint_a
    mint_b
    token_a_amount
    token_b_amount
    program
  }
}

```

### Response

The API responds with a JSON object containing a `data` field, which includes an array of pool data with the latest token balances.

```json
{
  "data": {
    "poolsData": [
      {
        "block_slot": 321249880,
        "block_time": "2025-02-17 12:17:04.741",
        "pool": "FcQRFJpQcdfyc4UHqMYMgzQTiy5wfxPvDcP9ywQN7SYR",
        "mint_a": "61V8vBaqAGMpgDQi4JcAwo1dmBGHsyhzodcPqnEVpump",
        "mint_b": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
        "token_a_amount": 86679.966529,
        "token_b_amount": 17380.573925
      }
    ]
  }
}

```



### Response Fields

* **`block_slot`**: The slot number of the block.
* **`block_time`**: The date time of the block.
* **`pool`**: The address of the pool.
* **`mint_a`**: The mint address for token A.
* **`mint_b`**: The mint address for token B.
* **`token_a_amount`**: The current balance of token A in the pool.
* **`token_b_amount`**: The current balance of token B in the pool.
* **`program`**: The address of the dex program.

### **Supported DEXes:**

For a list of supported DEXes and indexed data details, please refer to the [Supported DEXes and Indexed Data](https://www.notion.so/Supported-DEXes-and-Indexed-Data-14b2d9037ca78058ab32f8207665e53e?pvs=21) page
