---
icon: ghost
---

# Send Ghost Transaction

{% hint style="info" %}
Access is currently limited to invited users. Please create a ticket on Discord or contact us via Telegram to request access.
{% endhint %}

sendGhostTransaction allows you to submit a Solana transaction where the tip is supplied out of band, separately from the transaction itself.

Unlike a standard [sendTransaction](./#sendtransaction), the transaction need not contain a transfer to any one of [Astralane's tip wallets](../endpoints-and-configs/). Instead, the tip is supplied through a parameter while making the sendGhostTransaction request to one of our [endpoints](../endpoints-and-configs/).

> Key points:
>
> * Out of band tipping - tip is provided separately from the transaction
> * Supplying the tip in the request is mandatory, specified in lamports&#x20;
> * Smaller txn payload - no tip transfer needs to be embedded in the transaction
> * Same routing path - ghost transactions use the standard bundle router + dedicated iris pathways
> * Multiple transports - available through JSON-RPC, plain text, binary HTTP, and QUIC
> * Supports legacy and versioned transactions

## Send Ghost Transaction - Plain Text

The plain-text implementation uses the `/iris2` endpoint.

#### URI Params

<table><thead><tr><th width="171">Param</th><th width="120">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>api-key</code></td><td>String</td><td><strong>Mandatory</strong>, API key used for authentication</td></tr><tr><td><code>method</code></td><td>String</td><td><strong>Mandatory</strong>, must be <code>sendGhostTransaction</code> or <code>send_ghost_transaction</code></td></tr><tr><td><code>tip</code></td><td>u64</td><td><strong>Mandatory</strong>, out-of-band tip in lamports</td></tr><tr><td><code>mev-protect</code></td><td>Boolean</td><td><strong>Optional</strong>, enables MEV protection. Default is <code>false</code></td></tr><tr><td><code>swqos-only</code></td><td>Boolean</td><td><strong>Optional</strong>, if set to <code>true</code>, transaction is only sent through SWQoS. Default is <code>false</code></td></tr></tbody></table>

#### Example Request

The request body must contain the base64-encoded signed transaction.

The transaction itself must not contain a tip transfer.

```shellscript
// Some codecurl --location 'https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendGhostTransaction&tip=1000000' \
--header 'Content-Type: text/plain' \
--data 'base64_encoded_txn'
```

{% tabs %}
{% tab title="Rust" %}
```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::builder()
        .build()?;

    let api_key = "YOUR_API_KEY";
    let base64_tx = "base64_encoded_txn";
    let tip = 1_000_000u64;

    let url = format!(
        "https://lim.gateway.astralane.io/iris2?api-key={}&method=sendGhostTransaction&tip={}",
        api_key, tip
    );

    let response = client
        .post(url)
        .header("Content-Type", "text/plain")
        .body(base64_tx)
        .send()
        .await?;

    let body = response.text().await?;

    println!("{}", body);

    Ok(())
}
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import { Transaction, VersionedTransaction } from "@solana/web3.js";

/**
 * Sends a ghost transaction through the Astralane Iris2 gateway.
 *
 * The transaction must NOT contain an embedded tip transfer.
 * The tip is supplied separately in lamports.
 */
async function sendGhostTransaction(
  transaction: Transaction | VersionedTransaction,
  apiKey: string,
  tipLamports: number
): Promise<string> {
  const url = new URL("https://lim.gateway.astralane.io/iris2");

  url.searchParams.set("api-key", apiKey);
  url.searchParams.set("method", "sendGhostTransaction");
  url.searchParams.set("tip", tipLamports.toString());

  // Serialize and base64-encode the signed transaction.
  // The transaction must not contain a tip transfer.
  const base64Tx = Buffer.from(transaction.serialize()).toString("base64");

  const response = await fetch(url.toString(), {
    method: "POST",
    headers: {
      "Content-Type": "text/plain",
    },
    body: base64Tx,
  });

  const body = await response.json();

  if (!response.ok) {
    throw new Error(
      `sendGhostTransaction failed: ${response.status} ${JSON.stringify(body)}`
    );
  }

  return body.result;
}

// Usage
// ... build and sign your transaction ...
const signature = await sendGhostTransaction(
  signedTransaction,
  "YOUR_API_KEY",
  1_000_000
);

console.log(signature);
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const axios = require("axios");

const apiKey = "YOUR_API_KEY";
const tip = 1000000;
const base64Tx = "base64_encoded_txn";

const url =
  `https://lim.gateway.astralane.io/iris2` +
  `?api-key=${apiKey}` +
  `&method=sendGhostTransaction` +
  `&tip=${tip}`;

const config = {
  method: "post",
  maxBodyLength: Infinity,
  url,
  headers: {
    "Content-Type": "text/plain",
  },
  data: base64Tx,
};

axios
  .request(config)
  .then((response) => {
    console.log(JSON.stringify(response.data));
  })
  .catch((error) => {
    console.error(error.response?.data || error.message);
  });
```
{% endtab %}

{% tab title="Go Lang" %}
```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "strings"
)

func main() {
    apiKey := "YOUR_API_KEY"
    tip := "1000000"
    base64Tx := "base64_encoded_txn"

    url := fmt.Sprintf(
        "https://lim.gateway.astralane.io/iris2?api-key=%s&method=sendGhostTransaction&tip=%s",
        apiKey,
        tip,
    )

    payload := strings.NewReader(base64Tx)

    req, err := http.NewRequest("POST", url, payload)
    if err != nil {
        fmt.Println(err)
        return
    }

    req.Header.Add("Content-Type", "text/plain")

    client := &http.Client{}

    res, err := client.Do(req)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer res.Body.Close()

    body, err := io.ReadAll(res.Body)
    if err != nil {
        fmt.Println(err)
        return
    }

    fmt.Println(string(body))
}
```
{% endtab %}
{% endtabs %}

Here, `1000000` represents a tip of 0.001 SOL.

#### Expected Response

```json
{
  "result": "5KtPn1LGuxhFiwjxErkxTb7XxtLVjFW9QjMpv3Y7gW1x..."
}
```

## Send Ghost Transaction - Binary

The binary implementation uses the `/irisb` endpoint.

Unlike `/iris2`, the request body contains the raw serialized transaction bytes instead of a base64-encoded transaction.

#### URI Params

<table><thead><tr><th width="171">Param</th><th width="120">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>api-key</code></td><td>String</td><td><strong>Mandatory</strong>, API key used for authentication</td></tr><tr><td><code>method</code></td><td>String</td><td><strong>Mandatory</strong>, must be <code>sendGhostTransaction</code> or <code>send_ghost_transaction</code></td></tr><tr><td><code>tip</code></td><td>u64</td><td><strong>Mandatory</strong>, out-of-band tip in lamports</td></tr><tr><td><code>mev-protect</code></td><td>Boolean</td><td><strong>Optional</strong>, enables MEV protection</td></tr><tr><td><code>swqos-only</code></td><td>Boolean</td><td><strong>Optional</strong>, if set to <code>true</code>, transaction is only sent through SWQoS</td></tr></tbody></table>

#### Example Request

```shellscript
curl --location 'https://lim.gateway.astralane.io/irisb?api-key=APIKEY&method=sendGhostTransaction&tip=1000000' \
--header 'Content-Type: application/octet-stream' \
--data-binary @tx.bin
```

Where `tx.bin` contains a single serialized Solana transaction.

## Send Ghost Transaction - QUIC

Ghost transactions can be submitted over QUIC using a dedicated ghost-enabled QUIC listener.

> A standard QUIC listener cannot be used for ghost transactions.
>
> The target listener must be configured with `ghost: true`.
>
> For example:
>
> `:6000` - Ghost QUIC listener\
> `:7000` - Standard QUIC listener
>
> Sending a ghost frame to a standard QUIC listener will cause the frame to be interpreted as a normal transaction and dropped.

#### Wire Format

Each QUIC stream contains a single ghost frame:

```
┌──────────────────────────┬───────────────────────────────┐
│  tip: u64 little-endian  │   transaction bytes            │
│  8 bytes                 │   serialized transaction       │
└──────────────────────────┴───────────────────────────────┘
```

The first 8 bytes contain the tip in lamports encoded as a little-endian `u64`.

The remaining bytes contain the serialized Solana transaction.

#### Authentication

Authenticate with your `api_key`

```rust
let client = AstralaneQuicClient::connect(
    "host:6000",
    api_key
).await?;
```

#### Example

For a tip of `1,000,000` lamports:

```
tip.to_le_bytes()
    +
bincode::serialize(transaction)
```

The resulting bytes are sent as a single QUIC frame.

{% tabs %}
{% tab title="Rust" %}
```rust
let client = AstralaneQuicClient::connect(
    "host:6000",
    api_key
).await?;

let mut frame = tip.to_le_bytes().to_vec();

frame.extend_from_slice(
    &bincode::serialize(&tx)?
);

client.send_transaction(&frame).await?;

client.close().await;
```
{% endtab %}
{% endtabs %}

#### QUIC Response

QUIC ghost submissions are fire-and-forget.

There is no response body or transaction signature returned by the QUIC listener.

## Common Errors

If the transaction contains a recognized tip transfer, the request will be rejected:

```json
{
  "error": "ghost transaction must not embed a tip transfer to the tip account; pass the tip as a param"
}
```

If the `tip` parameter is missing:

```json
{
  "error": "missing required `tip` param"
}
```

| Condition                                      | /iris                                            | /iris2 / /irisb                      | QUIC          |
| ---------------------------------------------- | ------------------------------------------------ | ------------------------------------ | ------------- |
| Transaction contains a recognized tip transfer | JSON-RPC error                                   | `400` error                          | Frame dropped |
| Non-base64 encoding                            | `Only base64 encoded transactions are supported` | -                                    | -             |
| Missing tip                                    | Invalid request                                  | `400` - missing required `tip` param | -             |
| Method not recognized                          | `401`                                            | `401`                                | -             |
| Method not allowed                             | `405`                                            | `405`                                | -             |
| Ghost frame shorter than 8 bytes               | -                                                | -                                    | Frame dropped |
