---
description: 用于获取 DEX 池子最新余额的数据接口
---

# DEX 流动性池接口

**DEX 流动性池接口**提供面向各受支持去中心化交易所（DEX）池子的查询能力，可返回池子的详细信息与最新代币余额，便于开发者实时获取流动性数据并无缝集成到应用中。\
该 API 在设计上兼顾高性能与可扩展性，能快速、稳定地访问关键池子数据，是构建 DeFi 应用、分析平台与交易工具的基础组件。

**基础地址（Base URL）**\
[https://graphql.astralane.io/api/v1/dataset/dex-pools/v2](https://graphql.astralane.io/api/v1/dataset/dex-pools/v2)

### GraphQL Query 查询示例

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

### 响应示例

接口返回一个包含 data 字段的 JSON 对象，其中的 poolsData 数组包含各池子的最新代币余额。

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

### 字段说明

* **`block_slot`**: 区块的 slot 编号。
* **`block_time`**: 区块时间（日期时间）。
* **`pool`**: 池子地址。
* **`mint_a`**: 代币 A 的 mint 地址。
* **`mint_b`**: 代币 B 的 mint 地址。
* **`token_a_amount`**: 池中代币 A 的当前余额。
* **`token_b_amount`**: 池中代币 B 的当前余额。
* **`program`**: 对应 DEX 程序的地址。

### 支持的 DEX

关于已支持的 DEX 列表及索引的数据范围，请参见“[Supported DEXes and Indexed Data](https://www.notion.so/Supported-DEXes-and-Indexed-Data-14b2d9037ca78058ab32f8207665e53e?pvs=21)”页面。
