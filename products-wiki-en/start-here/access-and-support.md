---
description: What works straight away, what needs approval, and where to ask
icon: key
---

# Access and support

## What works with a new key

| Feature | Access |
| --- | --- |
| sendTransaction over JSON-RPC, plain text and binary | Straight away |
| QUIC and WebSocket | Straight away, on every tier |
| sendBundle | Every tier, at the tier's bundle limit |
| Tip stream | No key needed |
| Shreds | Through the portal, once your tips reach a shred tier |
| Astralane MCP | No key needed |

<!-- review: The fee tiers table gives the free tier 0 bundle TPS, while its FAQ says bundles work on the free tier. Confirm which is right before publishing this row. -->

## What needs approval

| Feature | How to ask | What to include |
| --- | --- | --- |
| Ghost transactions | Discord ticket or Telegram | Expected volume |
| Postpacks | Discord ticket | Your API key and region |
| Decoded shreds | Discord ticket | Region |
| sendBatch over JSON-RPC | Discord ticket | Batch size and volume |
| Tip refunds | Discord ticket | Your API key |
| Custom limits, private tip wallets, colocation | Discord or Telegram | Volume and current tier |

<!-- review: Add how long each approval usually takes. -->

## Where to ask

* **Discord:** [discord.gg/2UfWGtUDtN](https://discord.gg/2UfWGtUDtN). Open a ticket for access, limits and integration help.
* **Telegram:** [t.me/astralanecommunity](https://t.me/astralanecommunity)
* **Portal:** [portal.astralane.io](https://portal.astralane.io) for your key, tier and tip volume.

When something did not land, send the signature, the slot you expected, the region and the transport you used. [Troubleshooting](../help/troubleshooting.md) has the full checklist.
