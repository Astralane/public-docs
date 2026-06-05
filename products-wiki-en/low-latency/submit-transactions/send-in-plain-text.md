---
icon: fire
---

# Send in Plain Text

Key Improvements Include:

* No CORS preflight checks
* Plain text instead of a JSON payload
* Reduced body size, which means reduced bandwidth and network overhead

> Key points
>
> * Request should only have one header as `Content-Type : text/plain`
> * Txn should be in Base64 encoded
> * Compulsory to add `api-key` and `method` in URI params
> * Send request on endpoint `/iris2`

#### URI Params

<table><thead><tr><th width="171">Param</th><th width="120">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>api-key</code></td><td>String</td><td><strong>Mandatory</strong>, to set api key for authentication</td></tr><tr><td><code>method</code></td><td>String</td><td><strong>Mandatory</strong>, to set method</td></tr><tr><td><code>mev-protect</code></td><td>Boolean</td><td><strong>Optional</strong>, To set mev protect, default is <code>false</code></td></tr><tr><td><code>swqos-only</code></td><td>Boolean</td><td><strong>Optional</strong>, if set <code>true</code>, txn will be only send via swqos, default is <code>false</code></td></tr></tbody></table>

#### Methods

<table><thead><tr><th width="175">Method</th><th width="136">REST Method</th><th>Description</th></tr></thead><tbody><tr><td><code>sendTransaction</code></td><td><code>POST</code></td><td>To send base64 encoded transaction</td></tr><tr><td><code>sendBatch</code></td><td><code>POST</code></td><td>To send batch of up to 25 txns in Plain text</td></tr><tr><td><code>getHealth</code></td><td><code>POST</code></td><td>To keep alive connection</td></tr></tbody></table>

## Send Transaction

#### Example Request

```shellscript
curl --location 'https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction' \
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

    let mut headers = reqwest::header::HeaderMap::new();
    headers.insert("Content-Type", "text/plain".parse()?);

    let data = "base64_encoded_txn";

    let request = client.request(reqwest::Method::POST, "https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction")
        .headers(headers)
        .body(data);

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
 * Sends a base64-encoded Solana transaction as plain text
 * to the Astralane Iris2 gateway.
 */
async function sendTransaction(
  transaction: Transaction | VersionedTransaction,
  apiKey: string
): Promise<void> {
  const url = new URL("https://lim.gateway.astralane.io/iris2");
  url.searchParams.set("api-key", apiKey);
  url.searchParams.set("method", "sendTransaction");

  // Serialize and base64-encode the transaction
  const base64Tx = Buffer.from(transaction.serialize()).toString("base64");

  const response = await fetch(url.toString(), {
    method: "POST",
    headers: {
      "Content-Type": "text/plain",
    },
    body: base64Tx,
  });

  if (!response.ok) {
    throw new Error(
      `sendTransaction failed: ${response.status} ${response.statusText}`
    );
  }
}

// Usage
// ... build and sign your transaction ...
await sendTransaction(signedTransaction, "YOUR_API_KEY");
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const axios = require('axios');
let data = 'base64_encoded_txn';

let config = {
  method: 'post',
  maxBodyLength: Infinity,
  url: 'https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction',
  headers: { 
    'Content-Type': 'text/plain'
  },
  data : data
};

axios.request(config)
.then((response) => {
  console.log(JSON.stringify(response.data));
})
.catch((error) => {
  console.log(error);
});

```
{% endtab %}

{% tab title="Go Lang" %}
```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io"
)

func main() {

  url := "https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction"
  method := "POST"

  payload := strings.NewReader(`base64_encoded_txn`)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)

  if err != nil {
    fmt.Println(err)
    return
  }
  req.Header.Add("Content-Type", "text/plain")

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

## Send Batch

#### Example Request

```shellscript
curl --location 'https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendBatch' \
--header 'Content-Type: text/plain' \
--data 'base64_encoded_txn1,base64_encoded_txn2,base64_encoded_txn3' // coma seperated values
```

#### Expected Response

```json
{
  "result": ["sig1", "sig2" ....]
}
```

| Status Code | Description                    |
| ----------- | ------------------------------ |
| 200         | Request is successful          |
| 400         | Request might have some issues |
