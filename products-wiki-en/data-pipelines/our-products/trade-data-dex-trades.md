---
description: dex trades API
---

# Trade Data - Dex trades

The DEX Trades API parses all Solana-indexed transactions and allows users to query and fetch decentralized exchange (DEX) trades between specified block slots. Users can filter results by `program_id`, `start_block`, and `end_block` to retrieve detailed trade data.

**Base URL**

`POST` [`https://graphql.astralane.io/api/v1/dataset/dex/dex-trades/graphql`](https://graphql.astralane.io/api/v1/dataset/dex/dex-trades/graphql)

**Supported DEXes**&#x20;

[`https://audacelabs.notion.site/Supported-DEXes-and-Indexed-Data-14b2d9037ca78058ab32f8207665e53e?pvs=4`](https://audacelabs.notion.site/Supported-DEXes-and-Indexed-Data-14b2d9037ca78058ab32f8207665e53e?pvs=4)

<mark style="color:purple;background-color:yellow;">**Note**</mark><mark style="color:purple;background-color:yellow;">: The maximum supported block range per request is 100 blocks (e.g., from block</mark> <mark style="color:purple;background-color:yellow;"></mark><mark style="color:purple;background-color:yellow;">`317380000`</mark> <mark style="color:purple;background-color:yellow;"></mark><mark style="color:purple;background-color:yellow;">to</mark> <mark style="color:purple;background-color:yellow;"></mark><mark style="color:purple;background-color:yellow;">`317380100`</mark><mark style="color:purple;background-color:yellow;">).</mark>

### Data availability

**Earliest block** : `65304730` **February 14, 202**1

**Latest block**: Average delay of approximately 15 seconds from the latest block.

### Graphql Query

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



### Sample Response

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



### **Response Fields**

* **`block_time`**: Timestamp of the block.
* **`block_slot`**: Slot number of the block.
* **`tx_id`**: Transaction ID.
* **`signer`**: Signer of the transaction.
* **`pool_address`**: Address of the pool.
* **`base_mint`**: Base token mint address.
* **`quote_mint`**: Quote token mint address.
* **`base_vault`**: Base token vault address.
* **`quote_vault`**: Quote token vault address.
* **`base_amount`**: Amount of the base token involved.
* **`quote_amount`**: Amount of the quote token involved.
* **`is_inner_instruction`**: Whether the instruction is an inner instruction.
* **`instruction_index`**: Index of the instruction in the transaction.
* **`instruction_type`**: Type of the instruction.
* **`outer_program`**: Outer program ID.
* **`inner_program`**: Inner program ID (if applicable).
* **`txn_fee_lamports`**: Transaction fee in lamports.
* **`signer_lamports_change`**: Change in lamports for the signer.
* **`partition_0`**: Date partition for the transaction.
