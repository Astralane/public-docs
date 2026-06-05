---
description: >-
  Returns the top 100 wallets holding the specified token, ranked by their
  current token balance.
---

# Top 100 Token Holders

**Top 100 Token Holders API**

This API retrieves the top 100 holders of any SPL token, ranked by the current token balance in their wallets.

**Endpoint**\
**Base URL:** `https://graphql.astralane.io/api/v1/dataset`\
**GET** `/token/topHolders/<token-address>`

* **token-address:** The address of the SPL token (e.g., `SOL` or any valid SPL token address).

**Description:**\
Returns a list of the top 100 wallet addresses holding the specified SPL token, based on their current token balances. This is useful for token analytics, distribution insights, and whale tracking.

### Sample response

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

