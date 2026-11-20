---
description: Send one tipped transaction through Iris
icon: code
---

# sendTransaction

Sends one signed transaction. Iris routes it through staked connections, Jito, Harmonic and block builders at the same time, and returns the signature as soon as it accepts the transaction.

| | |
| --- | --- |
| **Endpoint** | `POST https://<region>.gateway.astralane.io/iris` |
| **Also on** | `/iris2` (plain text), `/irisb` (binary), QUIC, WebSocket |
| **Auth** | `api-key` URL parameter |
| **Limit** | 5 to 80 TPS, by tier. See [Pricing and limits](../help/pricing-and-limits.md) |
| **Minimum tip** | 0.001 SOL on the free tier, lower on VIP tiers. Token tips have their own minimums: see [Tips and fees](../send-transactions/tips-and-fees.md#minimums-for-token-tips) |

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
| 1 | object | No | `{ "encoding": "base64", "skipPreflight": true }`. Only base64 is accepted |
| 2 | object | No | `{ "mevProtect": true }`. Overrides `mev-protect` in the URL, including when set to `false` |

{% hint style="warning" %}
`mevProtect` is only read from the third item. Inside the second object it is ignored without an error.
{% endhint %}

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
| `-32600` | Invalid Request: min tip not paid for any asset | Tip below your minimum, sent to an address that is not a tip wallet, or in a form Iris does not count | Check the tip amount and the tip wallet |
| `-32600` | Invalid Request: Only base64 encoded transactions are supported | `encoding` is not `base64` | Send base64 |
| `-32600` | Invalid Request: failed to decode transaction only base64 is supported | Not valid base64, or too large | Check the encoding and the size |
| `-32600` | Invalid Request: invalid transaction … | The transaction cannot be parsed or a signature does not verify | Re-sign and re-serialize |
| `-32602` | | Missing or malformed `params` | Check the request shape above |
| `-32603` | Internal Error: failed to send | Iris could not forward the transaction | Retry once, then contact support |
| HTTP `401` | | Missing or unknown API key, or the method is not enabled on your key | Check the key in the portal; ask support to enable the method |
| HTTP `405` | | The method is not available on this path | Use the path listed for the method |
| HTTP `429` | | Over your per-second rate limit | Slow down or move up a tier |

Over plain text and binary, the same errors come back as HTTP `400` with `{"error": "<message>"}`. QUIC and WebSocket return nothing: invalid or over-limit transactions are dropped.

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

{% tab title="Rust" %}
```rust
use base64::{engine::general_purpose::STANDARD, Engine};
use serde_json::{json, Value};

// `tx` is your signed Transaction or VersionedTransaction
let url = format!(
    "https://fr.gateway.astralane.io/iris?api-key={}",
    std::env::var("ASTRALANE_API_KEY")?
);
let body = json!({
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [STANDARD.encode(bincode::serialize(&tx)?), { "encoding": "base64", "skipPreflight": true }],
});
let res: Value = reqwest::Client::new().post(url).json(&body).send().await?.json().await?;
let (signature, error) = (res["result"].as_str(), res.get("error"));
```
{% endtab %}

{% tab title="Python" %}
```python
import base64, os, requests

# `tx` is your signed solders Transaction or VersionedTransaction
url = f"https://fr.gateway.astralane.io/iris?api-key={os.environ['ASTRALANE_API_KEY']}"
res = requests.post(url, json={
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [base64.b64encode(bytes(tx)).decode(), {"encoding": "base64", "skipPreflight": True}],
}).json()
signature, error = res.get("result"), res.get("error")
```
{% endtab %}

{% tab title="Go" %}
```go
// `tx` is your signed *solana.Transaction (github.com/gagliardetto/solana-go)
raw, err := tx.MarshalBinary()
if err != nil {
	log.Fatal(err)
}
body, _ := json.Marshal(map[string]any{
	"jsonrpc": "2.0",
	"id":      1,
	"method":  "sendTransaction",
	"params":  []any{base64.StdEncoding.EncodeToString(raw), map[string]any{"encoding": "base64", "skipPreflight": true}},
})
url := "https://fr.gateway.astralane.io/iris?api-key=" + os.Getenv("ASTRALANE_API_KEY")
res, err := http.Post(url, "application/json", bytes.NewReader(body))
if err != nil {
	log.Fatal(err)
}
defer res.Body.Close()
var out struct {
	Result string          `json:"result"`
	Error  json.RawMessage `json:"error"`
}
if err := json.NewDecoder(res.Body).Decode(&out); err != nil {
	log.Fatal(err)
}
signature := out.Result
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

## Related

* [Bundles](../send-transactions/bundles.md) when several transactions must land together
* [Ghost transactions](../send-transactions/ghost-transactions.md) to pass the tip as a parameter
* [Troubleshooting](../help/troubleshooting.md) when a signature never lands
