---
icon: fire
---

# Send in Binary

Key Improvements Include:

1. No Base64 Encoding/Decoding Overhead
2. No Packet Splitting due to reduced data size
3. Reduced Transmission time

> Key points
>
> * Create Transaction in raw binary format
> * Header is optional
> * Mandatory to add `api-key` and `method` in URI params
> * Send request on endpoint `/irisb`&#x20;

## URI Params

<table><thead><tr><th width="171">Param</th><th width="120">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>api-key</code></td><td>String</td><td><strong>Mandatory</strong>, To set api key for authentication</td></tr><tr><td><code>method</code></td><td>String</td><td><strong>Mandatory</strong>, To set method</td></tr><tr><td><code>mev-protect</code></td><td>Boolean</td><td><strong>Optional</strong>, To set mev protect, default is <code>false</code></td></tr><tr><td><code>swqos-only</code></td><td>Boolean</td><td><strong>Optional</strong>, if set <code>true</code>, txn will be only send via swqos, default is <code>false</code></td></tr></tbody></table>

## Methods

<table><thead><tr><th width="175">Method</th><th width="136">REST Method</th><th>Description</th></tr></thead><tbody><tr><td><code>sendTransaction</code></td><td><code>POST</code></td><td>To send binary transaction</td></tr><tr><td><code>sendBatch</code></td><td><code>POST</code></td><td>To send batch of up to 25 txns in binary </td></tr><tr><td><code>getHealth</code></td><td><code>POST</code></td><td>To keep connection alive</td></tr></tbody></table>

## Send Transaction

#### Example Request

```shellscript
curl --location 'https://lim.gateway.astralane.io/irisb?api-key=APIKEY&method=sendTransaction' \
--data-binary @transaction.bin
```

Code Example :

{% tabs %}
{% tab title="Rust" %}
```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::builder()
        .build()?;

    let mut headers = reqwest::header::HeaderMap::new();
    headers.insert("Content-Type", "application/octet-stream".parse()?); //Optional

    let irisb_transaction_bytes = bincode::serialize(&irisb_transaction)
        .context("Failed to serialize IrisB transaction to binary")?;

    let request = client.request(reqwest::Method::POST, "https://lim.gateway.astralane.io/irisb?api-key=APIKEY&method=sendTransaction")
        .headers(headers)
        .body(irisb_transaction_bytes);

    let response = request.send().await?;
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
 * Sends a serialized Solana transaction as raw binary (application/octet-stream)
 * to the Astralane IrisB gateway.
 */
async function sendBinaryTransaction(
  transaction: Transaction | VersionedTransaction,
  apiKey: string
): Promise<void> {
  const txBytes: Uint8Array = transaction.serialize();

  const url = `https://<location>.gateway.astralane.io/irisb?api-key=${apiKey}&method=SendTransaction`;

  const response = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/octet-stream",
    },
    body: txBytes,
  });

  if (!response.ok) {
    throw new Error(
      `sendBinaryTransaction failed: ${response.status} ${response.statusText}`
    );
  }
}

// Usage
// ... build and sign your transaction ...
await sendBinaryTransaction(signedTransaction, "YOUR_API_KEY");
```
{% endtab %}
{% endtabs %}

#### Expected Response

```json
{
  "result": "5KtPn1LGuxhFiwjxErkxTb7XxtLVjFW9QjMpv3Y7gW1x..."
}
```

| Status Code | Description                    |
| ----------- | ------------------------------ |
| 200         | Request is successful          |
| 400         | Request might have some issues |

***

## Send Batch

#### Request Body Format

The body is a raw binary stream (`Content-Type: application/octet-stream`). Each transaction is framed as:

```
[ 2 bytes: tx length (big-endian u16) ][ N bytes: bincode-serialized transaction ]
```

Transactions are concatenated back-to-back with no separators or envelope:

```
+--------+----------+--------+----------+-----+
| len_0  |   tx_0   | len_1  |   tx_1   | ... |
| u16 BE | bincode  | u16 BE | bincode  |     |
+--------+----------+--------+----------+-----+
```

**Headers required:**

* `Content-Type: application/octet-stream`

#### Example

{% tabs %}
{% tab title="Rust" %}
```rust
use bincode;
use reqwest::Client;
use solana_sdk::{signature::Signature, transaction::Transaction};
use std::str::FromStr;

fn build_binary_body(transactions: &[Transaction]) -> Vec<u8> {
    let mut body = Vec::new();
    for tx in transactions {
        let tx_bytes = bincode::serialize(tx).expect("failed to serialize transaction");
        let len = tx_bytes.len() as u16;
        body.extend_from_slice(&len.to_be_bytes());
        body.extend_from_slice(&tx_bytes);
    }
    body
}

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let url = "https://lim.gateway.astralane.io/irisb?api-key=APIKEY&method=sendBatch";

    // Build your Vec<Transaction> however you like
    let transactions: Vec<Transaction> = vec![/* tx1, tx2, ... */];

    let binary_body = build_binary_body(&transactions);

    let client = Client::new();
    let response = client
        .post(url)
        .header("content-type", "application/octet-stream")
        .body(binary_body)
        .send()
        .await?;

    let status = response.status();
    let body_text = response.text().await?;

    if !status.is_success() {
        anyhow::bail!("request failed ({}): {}", status, body_text);
    }

    // Parse signatures from the response
    let parsed: serde_json::Value = serde_json::from_str(&body_text)?;

    let sig_array = parsed["result"]
        .as_array()
        .or_else(|| parsed.as_array())
        .expect("unexpected response format");

    let signatures: Vec<Signature> = sig_array
        .iter()
        .map(|v| {
            let s = v.as_str().expect("signature must be a string");
            Signature::from_str(s).expect("invalid signature")
        })
        .collect();

    for (i, sig) in signatures.iter().enumerate() {
        println!("tx[{}] signature: {}", i, sig);
    }

    Ok(())
}
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
import { Transaction } from "@solana/web3.js";

function buildBinaryBody(transactions: Transaction[]): Buffer {
  const chunks: Buffer[] = [];
  for (const tx of transactions) {
    const txBytes = tx.serialize();
    const lenBuf = Buffer.alloc(2);
    lenBuf.writeUInt16BE(txBytes.length, 0);
    chunks.push(lenBuf, txBytes);
  }
  return Buffer.concat(chunks);
}

async function sendBinaryBatch(
  url: string,
  transactions: Transaction[]
): Promise<string[]> {
  const body = buildBinaryBody(transactions);

  const response = await fetch(url, {
    method: "POST",
    headers: {
      "content-type": "application/octet-stream",
    },
    body,
  });

  if (!response.ok) {
    throw new Error(`request failed (${response.status}): ${await response.text()}`);
  }

  const json = await response.json();
  const signatures: string[] = json.result ?? json;

  signatures.forEach((sig, i) => console.log(`tx[${i}] signature: ${sig}`));
  return signatures;
}

// Usage
const transactions: Transaction[] = [/* tx1, tx2, ... */];
await sendBinaryBatch("https://lim.gateway.astralane.io/irisb?api-key=APIKEY&method=sendBatch", transactions);
```
{% endtab %}
{% endtabs %}

#### Response

The server returns JSON. The `result` field is an **array of signatures**, one per transaction in the order they were sent:

```json
{
  "result": [
    "5KtPn1LGuxhFiwjxErkxTb7XxtLVjFW9QjMpv3Y7gW1x...",
    "3RmHNpos7e4Bs3Y8j4Q1fGj5M5kPnxUeFNkMWv2jZrw5..."
  ]
}
```

* Array length matches the number of transactions sent.
* Signatures are in the same order as the transactions in the request body.
