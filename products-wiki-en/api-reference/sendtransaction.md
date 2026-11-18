---
description: Send one tipped transaction through Iris
icon: code
---

# sendTransaction

Sends one signed transaction. Iris routes it through staked connections, Jito, Paladin and block builders at the same time, and returns the signature as soon as it accepts the transaction.

| | |
| --- | --- |
| **Endpoint** | `POST https://<region>.gateway.astralane.io/iris` |
| **Also on** | `/iris2` (plain text), `/irisb` (binary), QUIC, WebSocket |
| **Auth** | `api-key` URL parameter |
| **Limit** | 5 to 80 TPS, by tier. See [Pricing and limits](../help/pricing-and-limits.md) |
| **Minimum tip** | 0.001 SOL on the free tier, lower on VIP tiers. Token tips count at their SOL value |

<!-- review: Auth is spelled four ways across the docs (api-key, api_key header, api_key on WebSocket, x-token). Pick one header for every service and keep the others working. -->

## URL parameters

| Name | Type | Required | Default | Meaning |
| --- | --- | --- | --- | --- |
| `api-key` | string | Yes | | Your key from the portal |
| `mev-protect` | boolean | No | `false` | See [MEV protection](../send-transactions/mev-protection.md) |
| `swqos-only` | boolean | No | `false` | Send only through SWQoS connections |

## Request

`params` is an array of up to three items.

| Position | Type | Required | Meaning |
| --- | --- | --- | --- |
| 0 | string | Yes | The signed transaction, base64 encoded, with a tip transfer in SOL, WSOL, USDC or USDT to an [Astralane tip wallet](../send-transactions/tips-and-fees.md#tip-wallets) |
| 1 | object | Yes | `{ "encoding": "base64", "skipPreflight": true }` |
| 2 | object | No | `{ "mevProtect": true }` |

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sendTransaction",
  "params": [
    "<base64 transaction>",
    { "encoding": "base64", "skipPreflight": true },
    { "mevProtect": true }
  ]
}
```

## Response

```json
{ "jsonrpc": "2.0", "id": 1, "result": "5VERv8NMhKqATzGBGFst5oBZoJN8sW2azHjfYFWRnMC9bYg2nFQrpRNr3zetGhKRKXnR8L1m7fBHU9bVNFRFcxk2" }
```

`result` is the transaction signature. It means Iris accepted the transaction. It does not mean the transaction landed: check with `getSignatureStatuses` on any RPC.

## Errors

| Code | Message | Cause | What to do |
| --- | --- | --- | --- |
| `-32600` | Transaction sanitization errors | Malformed transaction, or tip below your tier's minimum | Check the tip amount and the tip wallet |
| `-32600` | Only base64 encoded transactions are supported | Wrong encoding | Send base64 with `"encoding": "base64"` |
| `-32603` | Internal error | Iris could not forward the transaction | Retry once, then contact support |
| HTTP `401` | | Missing or unknown API key | Check the key in the portal |
| HTTP `429` | | Over your tier's rate limit | Slow down or move up a tier |

<!-- review: The codes above are the ones the current docs mention for sendBatch and ghost transactions. Engineering to confirm the full list for sendTransaction, including the error for a tip below the minimum and for a QoS throttle. -->

## Example

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const res = await fetch(`https://fr.gateway.astralane.io/iris?api-key=${process.env.ASTRALANE_API_KEY}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: 1,
    method: "sendTransaction",
    params: [tx.serialize().toString("base64"), { encoding: "base64", skipPreflight: true }],
  }),
});
const { result, error } = await res.json();
```
{% endtab %}

{% tab title="curl" %}
```bash
curl "https://fr.gateway.astralane.io/iris?api-key=$ASTRALANE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"sendTransaction","params":["<base64 transaction>",{"encoding":"base64","skipPreflight":true}]}'
```
{% endtab %}
{% endtabs %}

The full flow, from building the tip to checking the result, is in the [Quickstart](../start-here/quickstart.md).

<!-- review: Rust, Python and Go examples to come from the searcher-examples repo, built in CI. -->

## Related

* [Bundles](../send-transactions/bundles.md) when several transactions must land together
* [Ghost transactions](../send-transactions/ghost-transactions.md) to pass the tip as a parameter
* [Troubleshooting](../help/troubleshooting.md) when a signature never lands
