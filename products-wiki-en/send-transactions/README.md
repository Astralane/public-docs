---
description: Every way to send through Iris, and when to use each
icon: paper-plane
---

# Choose how to send

Iris takes a signed, tipped transaction and races it to the current leader over staked connections, Jito, Harmonic, block builders such as BAM, and custom schedulers such as Rakurai. You choose how the transaction reaches Iris.

## Transports

| Transport | Endpoint | Returns a signature | Best for |
| --- | --- | --- | --- |
| JSON-RPC | `https://<region>.gateway.astralane.io/iris` | Yes | Drop-in for any Solana library. Start here. |
| Plain text | `.../iris2?method=sendTransaction` | Yes | Browsers. No CORS preflight, smaller body. |
| Binary | `.../irisb?method=sendTransaction` | Yes | Backends that want to skip base64. |
| QUIC | `<region IP>:7000` | No | Bots that keep one connection open and send a lot. |
| WebSocket | `wss://<region>.gateway.astralane.io:5500/iris` | No | Browser terminals that want a persistent connection. |

{% hint style="warning" %}
QUIC and WebSocket return nothing. Rate-limited or invalid transactions are dropped without an error, so track your own signatures and landing rate.
{% endhint %}

## Methods

| Method | What it does | Page |
| --- | --- | --- |
| `sendTransaction` | One transaction, routed every way at once | [Reference](../api-reference/sendtransaction.md) |
| `sendBundle` | Up to 4 transactions that land together or not at all | [Bundles](bundles.md) |
| `sendIdeal` | Two versions of one transaction, one tip-heavy and one fee-heavy, sharing a durable nonce | [sendIdeal](send-ideal.md) |
| `sendBatch` | Up to 25 independent transactions in one request | [sendBatch](send-batch.md) |
| `sendGhostTransaction` | Tip passed as a parameter instead of inside the transaction | [Ghost transactions](ghost-transactions.md) |
| `getNonce`, `getHealth` | Managed nonce lookup and keep-alive | [sendIdeal](send-ideal.md), [Keep connections warm](keep-alive.md) |

## Options

| Option | Default | How to set it |
| --- | --- | --- |
| MEV protection | Off | See [MEV protection](mev-protection.md) |
| SWQoS only | Off | `swqos-only=true` in the URL. JSON-RPC, plain text and binary only |
| Retries | Set by Astralane | `maxRetries` in the `sendTransaction` config object. JSON-RPC only |
