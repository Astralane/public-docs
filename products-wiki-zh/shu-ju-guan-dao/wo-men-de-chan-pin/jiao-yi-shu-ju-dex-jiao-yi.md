---
description: DEX 交易数据接口
---

# 交易数据 - Dex 交易

DEX Trades API 会解析所有已索引的 Solana 交易，允许用户在指定的区块 slot 区间内查询并获取去中心化交易所（DEX）的成交记录。你可以通过 program\_id、start\_block 与 end\_block 过滤结果，以检索更细粒度的成交数据。

**基础地址（Base URL）**

`POST` [`https://graphql.astralane.io/api/v1/dataset/dex/dex-trades/graphql`](https://graphql.astralane.io/api/v1/dataset/dex/dex-trades/graphql)

**支持的 DEX 列表**

[`https://audacelabs.notion.site/Supported-DEXes-and-Indexed-Data-14b2d9037ca78058ab32f8207665e53e?pvs=4`](https://audacelabs.notion.site/Supported-DEXes-and-Indexed-Data-14b2d9037ca78058ab32f8207665e53e?pvs=4)

<mark style="color:purple;background-color:$warning;">**Note**</mark><mark style="color:purple;background-color:$warning;">: 注意：单次请求支持的最大区块范围为 100 个区块（例如从</mark> <mark style="color:purple;background-color:$warning;"></mark><mark style="color:purple;background-color:$warning;">`317380000`</mark> <mark style="color:purple;background-color:$warning;"></mark><mark style="color:purple;background-color:$warning;">到</mark> <mark style="color:purple;background-color:$warning;"></mark><mark style="color:purple;background-color:$warning;">`317380100`</mark><mark style="color:purple;background-color:$warning;">）。</mark>

### 数据可用性（Data availability）

**最早区块：**`65304730`（2021-02-14）

**最新区块：**&#x8F83;最新区块约 15 秒平均延迟

### 查询示例

```graphql
query DexTradeResult {
    dexTrades(
        start_block: 317380000
        end_block: 317380000
        program: "JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4"
    ) {
        tx_id
        block_time
        block_slot
        signer
        pool_address
        base_mint
        quote_mint
        base_vault
        quote_vault
        base_amount
        quote_amount
        is_inner_instruction
        instruction_index
        instruction_type
        inner_instruction_index
        outer_program
        inner_program
        txn_fee_lamports
    }
}
```

### **示例响应：**

```json
[
  {
    "block_time": "2024-08-13T00:04:09",
    "block_slot": 283239907,
    "tx_id": "3heh6YsGgM8WK1U1hEF7c26RKpMui2ZyeopmmQEX4PtwzWFRAadahJaVGWX31is9ufTHkEGN9GWwR1mxqencUS8c",
    "signer": "3eEuBoFRznqL6DUCVZpuWdJWeg7AfRdJSY3732PDSBuP",
    "pool_address": "CEGPBQJ6nYkDCiBP8AQ1LsPTj3bqhw34PKskzJiJgt6H",
    "base_mint": "So11111111111111111111111111111111111111112",
    "quote_mint": "3yvwMGa3ap49DWquMEpf7Mrvq85N4gjXsdehFPRRpump",
    "base_vault": "BW8fXFoEGbhRf1LpdX53Ems3BHWdCNaTdyAR6XxmpNQa",
    "quote_vault": "7xynyUq3VVAKVgBbgnb4ssofcMpX3EiRsiadFWsc2XmR",
    "base_amount": -0.000016546,
    "quote_amount": 87.64461,
    "is_inner_instruction": false,
    "instruction_index": 4,
    "instruction_type": "SwapBaseIn",
    "inner_instruxtion_index": 0,
    "outer_program": "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8",
    "inner_program": "",
    "txn_fee_lamports": 23990,
    "signer_lamports_change": -23990,
    "partition_0": "2024-08-13"
  }
]
```

### 字段说明（Response Fields）

* **`block_time`**：区块时间戳。
* **`block_slot`**：区块的 slot 编号。
* **`tx_id`**：交易哈希。
* **`signer`**：交易签名者地址。
* **`pool_address`**：池子地址。
* **`base_mint`**：基础代币（base）mint 地址。
* **`quote_mint`**：计价代币（quote）mint 地址。
* **`base_vault`**：基础代币金库地址。
* **`quote_vault`**：计价代币金库地址。
* **`base_amount`**：本次成交涉及的基础代币数量。
* **`quote_amount`**：本次成交涉及的计价代币数量。
* **`is_inner_instruction`**：是否为内部指令（inner instruction）。
* **`instruction_index`**：该指令在交易中的索引。
* **`instruction_type`**：指令类型。
* **`outer_program`**：外层程序 ID。
* **`inner_program`**：内层程序 ID（如适用）。
* **`txn_fee_lamports`**：交易手续费（以 lamports 计）。
* **`signer_lamports_change`**：签名者的 lamports 变动。
* **`partition_0`**：按日期分区标记。
