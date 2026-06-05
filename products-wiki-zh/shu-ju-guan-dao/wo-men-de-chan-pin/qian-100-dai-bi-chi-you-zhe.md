---
description: 返回当前持有指定代币余额最高的前 100 个钱包地址，并按余额排序。
---

# 前 100 代币持有者

**前 100 代币持有者API**\
该 API 用于获取任意 SPL 代币的前 100 名持币者，依据其钱包中的当前代币余额进行排名。

**Endpoint**

**基础地址（Base URL）**：[`https://graphql.astralane.io/api/v1/dataset`](https://graphql.astralane.io/api/v1/dataset)

**GET** `/token/topHolders/<token-address>`

* **token-address：**&#x53;PL 代币地址（例如 SOL 或任意有效的 SPL 代币地址）。

**描述：**\
基于当前代币余额，返回持有指定 SPL 代币的前 100 个钱包地址列表。适用于代币分析、持币分布洞察和大户（whale）追踪等场景。

### **示例响应：**

```javascript
{
        "block_slot": 326645617,
        "mint": "DCXqS9xMQqoD2cRpqvyu2TxpherRwtd2UDHLKTk7pump",
        "owner": "5Q544fKrFoe6tsEbD7S8EmxGTJYAKtTVhAW5Q5pge4j1",
        "post_balance": 151282094.752046,
        "token_account": "Gru1x7otg6Sr5wcZU2LAaqBQ7XdvRYMDJdmwhiZvxmzA",
        "partition_0": "2025-03-14"
    },
{
        "block_slot": 324358055,
        "mint": "DCXqS9xMQqoD2cRpqvyu2TxpherRwtd2UDHLKTk7pump",
        "owner": "FevmaA1vYkUkCJUMfVjGMfLHkKBTatD7gyP671Smy46p",
        "post_balance": 22719427.69204,
        "token_account": "7p5Lyqa7uci7Q55pFsxRLhoT9snRQKT6du58DArv3TPe",
        "partition_0": "2025-03-03"
    }
```

