---
description: What MEV protection does, and how to turn it on for each transport
icon: shield-halved
---

# MEV protection

MEV protection is off by default.

<!-- review: The current docs never say what mev-protect does. Engineering to fill the next section: which routes are skipped, what it costs in speed or landing rate, and whether it is a guarantee or a reduction. -->

## What it does

{% hint style="info" %}
Draft, for engineering to complete: which routes Iris skips when MEV protection is on, what that costs in speed or landing rate, and whether it removes sandwich risk or only reduces it.
{% endhint %}

## Turn it on

| Transport | How |
| --- | --- |
| JSON-RPC, plain text, binary | Add `mev-protect=true` to the URL |
| JSON-RPC, per request | Pass `{ "mevProtect": true }` as the third parameter of `sendTransaction`, or inside the options of `sendBundle` and `sendBatch` |
| QUIC | Connect to port `9000` instead of `7000` |
| WebSocket | Connect to the `/mev-protect` path instead of `/iris` |

<!-- review: If the URL parameter and the request parameter disagree, which one wins? -->

## Extra protection for bundles

Add any valid Solana public key that starts with `jitodontfront` or `dontbund1e` to a transaction. Jito and Harmonic block engines then reject any bundle that places that transaction anywhere except first. See [Jito](https://docs.jito.wtf/lowlatencytxnsend/#sandwich-mitigation) and [Harmonic](https://docs.harmonic.gg/searchers/bundle-control-accounts).

## Check a transaction afterwards

The [Astralane MCP](../astralane-mcp.md) tool `astralane_investigate_sandwich` checks a signature for front-running and sandwiching.
