---
description: What works straight away, what needs approval, and where to ask
icon: key
---

# Access and support

## What works with a new key

| Feature | Access |
| --- | --- |
| sendTransaction over JSON-RPC, plain text and binary | Straight away, at 5 TPS |
| sendIdeal and getNonce over JSON-RPC | Straight away, at 5 TPS |
| getHealth | Straight away |
| [GBX](../gravity-block-exchange-gbx/submit-bundles.md) public bundle submission | No key needed, at public rate limits |
| Tip stream | No key needed |
| Shreds | Through the portal, once your tips reach a shred tier |
| Astralane MCP | No key needed |

Everything else is switched off on a new key and is enabled per key: see below. Over HTTP, calling a method that is not enabled returns `401`; over QUIC and WebSocket the transactions are dropped.

## What needs approval

| Feature | How to ask | What to include |
| --- | --- | --- |
| sendBundle (included from VIP 1) | Discord ticket | Expected volume and bundle success rate |
| GBX authenticated access (higher limits) | Discord ticket | Your searcher pubkey and expected volume |
| QUIC and WebSocket | Discord ticket | Expected volume and region |
| Ghost transactions | Discord ticket or Telegram | Expected volume |
| Postpacks | Discord ticket | Your API key and region |
| Decoded shreds | Discord ticket | Region |
| sendBatch | Discord ticket | Batch size and volume |
| Tip refunds | Discord ticket | Your API key |
| Custom limits, private tip wallets, colocation | Discord or Telegram | Volume and current tier |

The Astralane team reviews and approves each request.

## Where to ask

* **Discord:** [discord.gg/2UfWGtUDtN](https://discord.gg/2UfWGtUDtN). Open a ticket for access, limits and integration help.
* **Telegram:** [t.me/astralanecommunity](https://t.me/astralanecommunity)
* **Portal:** [portal.astralane.io](https://portal.astralane.io) for your key, tier and tip volume.

When something did not land, send the signature, the slot you expected, the region and the transport you used. [Troubleshooting](../help/troubleshooting.md) has the full checklist.
