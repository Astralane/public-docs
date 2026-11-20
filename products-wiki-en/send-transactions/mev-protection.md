---
description: What MEV protection does, and how to turn it on for each transport
icon: shield-halved
---

# MEV protection

MEV protection is off by default.

## What it does

MEV protection prevents toxic MEV, such as sandwiching.

## Turn it on

| Transport | How |
| --- | --- |
| JSON-RPC, plain text, binary | Add `mev-protect=true` to the URL. JSON-RPC `sendBundle` and `sendBatch` ignore it: use the request option below |
| JSON-RPC, per request | Pass `{ "mevProtect": true }` as the third parameter of `sendTransaction`, or inside the options of `sendBundle` and `sendBatch` |
| QUIC | Connect to port `9000` instead of `7000` |
| WebSocket | Connect to the `/mev-protect` path instead of `/iris` |

If the URL and the request disagree, the request wins, including `"mevProtect": false`. On QUIC and WebSocket, protection is set by the port or path, not per transaction.

## Extra protection for bundles

Add any valid Solana public key that starts with `jitodontfront` or `dontbund1e` to a transaction. Jito and Harmonic block engines then reject any bundle that places that transaction anywhere except first. See [Jito](https://docs.jito.wtf/lowlatencytxnsend/#sandwich-mitigation) and [Harmonic](https://docs.harmonic.gg/searchers/bundle-control-accounts).

## Check a transaction afterwards

The [Astralane MCP](../astralane-mcp.md) tool `astralane_investigate_sandwich` checks a signature for front-running and sandwiching.
