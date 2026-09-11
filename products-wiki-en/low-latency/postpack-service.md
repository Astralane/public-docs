---
description: The fastest transaction feed on Solana.
icon: box-dollar
---

# Postpack Service

### What are postpacks?

A post-pack is the leader's notification that a transaction has passed the point of no return in its execution path. The transaction hasn't been executed yet, but its account locks are already held, so no conflicting transaction can be ordered ahead of it. Under normal conditions, it can no longer be front-run. Post-packs are currently the fastest way to learn that a transaction is being include in a Solana block.

<table><thead><tr><th width="162.29296875"></th><th>Postpacks</th><th>Preconfs</th><th>Shreds</th></tr></thead><tbody><tr><td><strong>Emitted at</strong></td><td>Right <strong>before tx</strong> <strong>execution</strong>, after account locks are held</td><td>Right <strong>after</strong> <strong>tx</strong> <strong>execution</strong>, before including into the block.</td><td><strong>After including</strong> tx into the block.</td></tr><tr><td><strong>Execution Result</strong></td><td>Absent</td><td>Present</td><td>Present</td></tr><tr><td><strong>Tx index in the block</strong></td><td>Absent</td><td>Present</td><td>Present</td></tr><tr><td><strong>Landing probability</strong></td><td>Lower</td><td>High</td><td>High</td></tr><tr><td><strong>Latency</strong></td><td>High</td><td>Lower</td><td>Slowest</td></tr></tbody></table>

### Connection

Endpoint address: [http://fr.relay.astralane.io:30001](http://fr.relay.astralane.io:30001)

Auth: use `x-token` header with your API key to connect.

Protocol: same as original harmonic preconfs proto: [https://github.com/Astralane/postpack-conf-client/blob/main/protos/preconf.proto](https://github.com/Astralane/postpack-conf-client/blob/main/protos/preconf.proto)

Use our example client lib to get started: [https://github.com/Astralane/postpack-conf-client](https://github.com/Astralane/postpack-conf-client)

{% hint style="info" %}
You need to get whitelisted before you can use your API key for preconfs. Create ticket on [Discord](https://discord.gg/2UfWGtUDtN) to request access.&#x20;
{% endhint %}

### Pricing

PostpackPay is a separate HTTP service used only for registering tips - not for landing transactions.\
You send:

* signed transaction;
* slot that triggered the transaction.

**Request format:**

`POST` to your endpoint URL, JSON body (array):

```http
...
POST /postpack-pay?api-key=<your-api-key> HTTP/1.1
Host: edge.astralane.io
Content-Type: application/json

[{ "transaction": "<base64 bincode-encoded signed tx>", "slot": 123456789 }]
...
```

**Response format:**

```json
{ "accepted": ["<your transaction signature>"] }
```

You can land your transactions anywhere — Iris or any other provider. That part is entirely up to you.\


{% hint style="info" icon="info" %}
A tip only counts once a copy of that transaction is sent to **PostpackPay** over HTTP. Landing it somewhere doesn't register it — not even through Astralane's own services.
{% endhint %}

#### How to set up tipping&#x20;

The best use of such model is integrating tips into your searcher code:

```
// main searcher logic
...
if astralane_postpack_was_a_source_signal() {
    land_transaction()              // any provider
    send_tx_copy_to_postpackpay()     // registers the tip
}
```

#### Tipping Address

* astgCFnATW3DGN7Pnfj51w3BdVy2TGeMoK5eV7bt4x1
* astk6J92kctKFznK4SD3jaKCSTKftSPUV4W2r529HUx
* ast5CxenCFBBv99pf8hhJSTG7V3oeCTLqpRrcLYj9Bs
* asteY6wTGTTQ8amUUa7xRxf4Nc3kGV9ritjX6yJLFi3

#### ShredPay endpoints

Send your [POST request](postpack-service.md#how-your-tips-get-counted) to the endpoint closest to your infra:

<table><thead><tr><th width="272.22222900390625">Region</th><th>HTTP Endpoint</th></tr></thead><tbody><tr><td>Global Edge Endpoint</td><td><a href="https://edge.astralane.io/postpack-pay?api-key=xxxx">https://edge.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>Frankfurt (Recommended)</td><td><a href="http://fr.gateway.astralane.io/postpack-pay?api-key=xxxx">http://fr.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>Frankfurt</td><td><a href="http://fr2.gateway.astralane.io/postpack-pay?api-key=xxxx">http://fr2.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>San Francisco</td><td><a href="http://la.gateway.astralane.io/postpack-pay?api-key=xxxx">http://la.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>Tokyo</td><td><a href="http://jp.gateway.astralane.io/postpack-pay?api-key=xxxx">http://jp.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>New York</td><td><a href="http://ny.gateway.astralane.io/postpack-pay?api-key=xxxx">http://ny.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>Amsterdam (Recommended)</td><td><a href="http://ams.gateway.astralane.io/postpack-pay?api-key=xxxx">http://ams.gateway.astralane.io/postpack-pay?api-key=xxx</a></td></tr><tr><td>Amsterdam 2</td><td><a href="http://ams2.gateway.astralane.io/postpack-pay?api-key=xxxx">http://ams2.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>Limburg</td><td><a href="http://lim.gateway.astralane.io/postpack-pay?api-key=xxxx">http://lim.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>Singapore</td><td><a href="http://sg.gateway.astralane.io/postpack-pay?api-key=xxxx">http://sg.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr><tr><td>Lithuania</td><td><a href="http://lit.gateway.astralane.io/postpack-pay?api-key=xxxx">http://lit.gateway.astralane.io/postpack-pay?api-key=xxxx</a></td></tr></tbody></table>
