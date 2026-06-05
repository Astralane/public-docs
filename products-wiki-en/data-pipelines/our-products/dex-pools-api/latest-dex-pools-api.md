---
description: >-
  The DEX Pools API allows you to retrieve information about the latest
  liquidity pools created across various decentralized exchanges (DEX) on the
  Solana blockchain.
---

# Latest Dex Pools API

**GraphQL API Endpoint**

Make requests using the following endpoint:

`POST https://graphql.astralane.io/api/v1/dataset/latest-pools`&#x20;



**Request Headers**

| Header      | Description                               | Required |
| ----------- | ----------------------------------------- | -------- |
| `x-api-key` | Your API key provided for authentication. | Yes      |



**Graphql Query**

```json
query PoolsData {
    poolsData {
        block_slot
        blockTime
        pool
        mint_a
        mint_b
        token_a_amount
        token_b_amount
    }
}
```

#### Response Example

A successful response will contain a JSON array with details of the latest pools and their token balances:

```json
{
    "data": {
        "poolsData": [
            {
                "block_slot": 303089869,
                "blockTime": null,
                "pool": "8dM6UzQAq2fNaEtfQr6nNzavpTtywcE1GNBXhmGGE8rd",
                "mint_a": "So11111111111111111111111111111111111111112",
                "mint_b": "31QC34eES9xy7BZmptDgeRy5Mi1iJ51FcXszTj49qwPX",
                "token_a_amount": 30.328900364,
                "token_b_amount": 165210214.980515
            },
            {
                "block_slot": 303089853,
                "blockTime": null,
                "pool": "2o6xd3dYguhofKBT6xUNXHQggiKbSEv5vZ9LMiuymquD",
                "mint_a": "So11111111111111111111111111111111111111112",
                "mint_b": "4yzCocQbtUpd6uanGPp4nxZYiAg8Yv3aCjEMysiMpump",
                "token_a_amount": 0.012082268,
                "token_b_amount": 23547.163168
            }
            ]
    }
         


```

#### Response Fields

* **`block_time`**: Timestamp indicating when the block was processed.
* **`block_slot`**: Slot number associated with the processed block.
* **`pool`**: Unique address identifier for the liquidity pool.
* **`mint_a`**: Mint address of the base token.
* **`mint_b`**: Mint address of the quote token.
* **`token_a_amount`**: Quantity of the base token in the pool.
* **`token_b_amount`**: Quantity of the quote token in the pool.

#### API Data Refresh Interval

The API indexes and updates data every **30 minutes**, capturing all new pools created within this interval. This periodic indexing ensures pool data accuracy and reliability, balancing timeliness and performance.

Ensure that your requests always include the correct API key in the headers for seamless access to the service.
