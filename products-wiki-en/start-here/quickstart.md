---
description: Land your first transaction through Iris in five steps
icon: bullseye-arrow
---

# Quickstart

This guide sends one tipped transaction through Iris over JSON-RPC and checks that it landed. It takes about ten minutes.

{% stepper %}
{% step %}
### Get an API key

Create an account at [portal.astralane.io](https://portal.astralane.io) and copy your API key. A new key starts on the free tier: 5 transactions per second and a minimum tip of 0.001 SOL.

{% hint style="info" %}
Keep the key out of code you commit. The examples read it from `ASTRALANE_API_KEY`.
{% endhint %}
{% endstep %}

{% step %}
### Pick the endpoint nearest your servers

Use the region closest to where your bot runs. If you are not sure, start with Frankfurt.

| Region | JSON-RPC endpoint |
| --- | --- |
| Frankfurt | `https://fr.gateway.astralane.io/iris` |
| Amsterdam | `https://ams.gateway.astralane.io/iris` |
| New York | `https://ny.gateway.astralane.io/iris` |
| Tokyo | `https://jp.gateway.astralane.io/iris` |

All eleven regions, with IPs and QUIC ports, are in [Regions and endpoints](../send-transactions/regions-and-endpoints.md).
{% endstep %}

{% step %}
### Add a tip to your transaction

Iris only forwards transactions that pay a tip to one of the [Astralane tip wallets](../send-transactions/tips-and-fees.md#tip-wallets). Add a SOL transfer as an instruction in the same transaction, at or above your tier's minimum. You can also tip in WSOL, USDC or USDT; see [Tip in WSOL, USDC or USDT](../send-transactions/tips-and-fees.md#tip-in-wsol-usdc-or-usdt).

```typescript
import {
  ComputeBudgetProgram, Connection, Keypair, PublicKey, SystemProgram, Transaction,
} from "@solana/web3.js";

const TIP_WALLET = new PublicKey("astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF");
const TIP_LAMPORTS = 1_000_000; // 0.001 SOL, the free-tier minimum

const payer = Keypair.fromSecretKey(/* your key */);
const rpc = new Connection("https://api.mainnet-beta.solana.com", "confirmed"); // any RPC, used for the blockhash
const latest = await rpc.getLatestBlockhash();

const tx = new Transaction({ feePayer: payer.publicKey, ...latest }).add(
  ComputeBudgetProgram.setComputeUnitPrice({ microLamports: 10_000 }),
  // ...your own instructions...
  SystemProgram.transfer({ fromPubkey: payer.publicKey, toPubkey: TIP_WALLET, lamports: TIP_LAMPORTS }),
);
tx.sign(payer);
```

{% hint style="warning" %}
A tip below your tier's minimum is rejected. Pick a different tip wallet for each send, so your transactions do not queue on the same account lock.
{% endhint %}
{% endstep %}

{% step %}
### Send it

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const url = `https://fr.gateway.astralane.io/iris?api-key=${process.env.ASTRALANE_API_KEY}`;

const res = await fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: 1,
    method: "sendTransaction",
    params: [tx.serialize().toString("base64"), { encoding: "base64", skipPreflight: true }],
  }),
});
const { result: signature, error } = await res.json();
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
  -d '{"jsonrpc":"2.0","id":1,"method":"sendTransaction",
       "params":["<base64 transaction>",{"encoding":"base64","skipPreflight":true}]}'
```
{% endtab %}
{% endtabs %}

A `result` is the transaction signature. It means Iris accepted the transaction, not that it landed.

{% endstep %}

{% step %}
### Check that it landed

```typescript
const { value } = await rpc.getSignatureStatuses([signature]);
console.log(value[0]?.confirmationStatus); // "processed", "confirmed" or "finalized"
```

If the status is still empty after the blockhash expires, the transaction did not land. [Troubleshooting](../help/troubleshooting.md) lists the causes in the order to check them.
{% endstep %}
{% endstepper %}

## Next

* Send faster by keeping a connection open: [QUIC](../send-transactions/quic.md) or [WebSocket](../send-transactions/websocket.md).
* Send several transactions that must land together: [Bundles](../send-transactions/bundles.md).
* Pay less once your volume grows: [Pricing and limits](../help/pricing-and-limits.md).
