---
description: Every Iris method, with its transports and where it is documented
icon: book
---

# All methods

Every method page follows the same shape: endpoint, auth, limits, parameters, request, response, errors and an example. [sendTransaction](sendtransaction.md) is the first one written this way.

| Method | What it does | Transports | Access |
| --- | --- | --- | --- |
| [sendTransaction](sendtransaction.md) | One tipped transaction | JSON-RPC, plain text, binary, QUIC, WebSocket | Every tier. QUIC and WebSocket on request |
| [sendBundle](../send-transactions/bundles.md) | Up to 4 transactions, all or nothing | JSON-RPC | VIP 1 and above, or on request |
| [sendIdeal](../send-transactions/send-ideal.md) | A tip-heavy and a fee-heavy version, sharing a durable nonce | JSON-RPC | Every tier |
| [getNonce](../send-transactions/send-ideal.md) | Returns your managed nonce account and current nonce | JSON-RPC | Every tier |
| [sendBatch](../send-transactions/send-batch.md) | Up to 25 independent transactions | JSON-RPC, plain text, binary | On request |
| [sendGhostTransaction](../send-transactions/ghost-transactions.md) | Tip passed as a parameter | JSON-RPC, plain text, binary, QUIC | On request |
| getHealth | Keeps a connection warm | JSON-RPC, `/gethealth`, plain text, binary | Every tier |
