---
description: What works straight away, what needs approval, and where to ask
icon: key
---

# Access and support

## What works with a new key

| Feature | Access |
| --- | --- |
| sendTransaction and sendIdeal over JSON-RPC, plain text and binary | Straight away, at 5 TPS |
| Tip stream | No key needed |
| Shreds | Through the portal, once your tips reach a shred tier |
| Astralane MCP | No key needed |

Everything else is switched off on a new key and is enabled per key: see below. Over HTTP, calling a method that is not enabled returns `401`; over QUIC and WebSocket the transactions are dropped.

## What needs approval

| Feature | How to ask | What to include |
| --- | --- | --- |
| sendBundle | Discord ticket | Expected volume and bundle success rate |
| QUIC and WebSocket | Discord ticket | Expected volume and region |
| Ghost transactions | Discord ticket or Telegram | Expected volume |
| Postpacks | Discord ticket | Your API key and region |
| Decoded shreds | Discord ticket | Region |
| sendBatch over JSON-RPC | Discord ticket | Batch size and volume |
| Tip refunds | Discord ticket | Your API key |
| Custom limits, private tip wallets, colocation | Discord or Telegram | Volume and current tier |

The Astralane team reviews and approves each request.

## Where to ask

* **Discord:** [discord.gg/2UfWGtUDtN](https://discord.gg/2UfWGtUDtN). Open a ticket for access, limits and integration help.
* **Telegram:** [t.me/astralanecommunity](https://t.me/astralanecommunity)
* **Portal:** [portal.astralane.io](https://portal.astralane.io) for your key, tier and tip volume.

When something did not land, send the signature, the slot you expected, the region and the transport you used. [Troubleshooting](../help/troubleshooting.md) has the full checklist.
