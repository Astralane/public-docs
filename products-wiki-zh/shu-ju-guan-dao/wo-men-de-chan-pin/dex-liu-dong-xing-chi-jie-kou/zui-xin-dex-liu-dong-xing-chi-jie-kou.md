---
description: 用于获取 Solana 链上各主流 DEX 新创建的最新流动性池信息。
---

# 最新Dex流动性池接口

**GraphQL API Endpoint**

使用以下端点发起请求（POST）：

`POST https://graphql.astralane.io/api/v1/dataset/latest-pools`&#x20;



**请求头**

| **Header**  | **描述**          | **必要性** |
| ----------- | --------------- | ------- |
| `x-api-key` | 用于身份认证的 API Key | 是       |



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

#### 响应示例

成功响应将返回包含最新池子及其代币余额信息的 JSON：

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

#### 字段说明

* **`block_time`**: 区块处理时间戳。
* **`block_slot`**: 区块对应的 slot 编号。
* **`pool`**: 流动性池的唯一地址。
* **`mint_a`**: 基础代币（base token）的 mint 地址。
* **`mint_b`**: 计价代币（quote token）的 mint 地址。
* **`token_a_amount`**: 池中基础代币数量。
* **`token_b_amount`**: 池中计价代币数量。



**数据刷新频率**\
API **每 30 分钟**进行一次索引与更新，在该时间窗口内捕获所有新创建的池子。该周期性索引在及时性与性能之间实现平衡，保障数据的准确性与可靠性。

**注意：**&#x4E3A;确保正常访问，请在请求头中正确携带 x-api-key。
