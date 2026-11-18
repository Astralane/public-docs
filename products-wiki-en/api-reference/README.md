---
description: Every Iris method, with its transports and where it is documented
icon: book
---

# All methods

Every method page follows the same shape: endpoint, auth, limits, parameters, request, response, errors and an example. [sendTransaction](sendtransaction.md) is the first one written this way.

| Method | What it does | Transports | Access |
| --- | --- | --- | --- |
| [sendTransaction](sendtransaction.md) | One tipped transaction | JSON-RPC, plain text, binary, QUIC, WebSocket | Every tier |
| [sendBundle](../send-transactions/bundles.md) | Up to 4 transactions, all or nothing | JSON-RPC | Every tier, at the tier's bundle limit |
| [sendIdeal](../send-transactions/send-ideal.md) | A tip-heavy and a fee-heavy version, sharing a durable nonce | JSON-RPC | Every tier |
| getNonce | Returns your managed nonce account and current nonce | JSON-RPC | Every tier |
| sendBatch | Up to 25 independent transactions | JSON-RPC (on request), plain text, binary | On request for JSON-RPC |
| [sendGhostTransaction](../send-transactions/ghost-transactions.md) | Tip passed as a parameter | Plain text, binary, QUIC | Invited users |
| getHealth | Keeps a connection warm | JSON-RPC, `/gethealth`, plain text, binary | Every tier |
| sendPaladin | Sends through Paladin | JSON-RPC | |

<!-- review: sendPaladin is listed as an available call but documented nowhere. getNonce and sendBatch need their own pages in the sendTransaction shape. -->
