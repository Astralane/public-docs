---
description: Up to 25 independent transactions in one request
icon: list
---

# sendBatch

Submits a batch of serialized transactions for processing. Returns the transaction signatures upon successful submission.

> Note :&#x20;
>
> Only available on request

### Request

**Method:** `sendBatch`

#### Parameters

<table><thead><tr><th width="117">Type</th><th width="105">Required</th><th>Description</th></tr></thead><tbody><tr><td><code>string[]</code></td><td>Yes</td><td>Array of base64-encoded serialized transactions. Maximum batch size is <strong>25</strong> transactions. Each transaction must include a valid tip.</td></tr><tr><td><code>object</code></td><td>No</td><td>Optional configuration object.</td></tr></tbody></table>

**Configuration Object**

<table><thead><tr><th width="119">Field</th><th width="102">Type</th><th width="107">Default</th><th>Description</th></tr></thead><tbody><tr><td><code>mevProtect</code></td><td><code>boolean</code></td><td><code>false</code></td><td>When <code>true</code>, enables MEV protection for all transactions in the batch.</td></tr></tbody></table>

### Response

Returns `string[]` — an array of base58-encoded transaction signatures, in the same order as the input transactions.

### Errors

<table><thead><tr><th width="98">Code</th><th>Message</th><th>Description</th></tr></thead><tbody><tr><td><code>-32600</code></td><td><code>request size {n} exceeded max batch size {max}</code></td><td>Batch exceeds the maximum allowed size.</td></tr><tr><td><code>-32600</code></td><td><code>contains duplicate signature</code></td><td>Two or more transactions in the batch share the same signature.</td></tr><tr><td><code>-32600</code></td><td><code>cannot validate transaction ...</code></td><td>A transaction is malformed or does not meet the minimum tip requirement.</td></tr><tr><td><code>-32603</code></td><td><code>failed to send batch, internal error</code></td><td>Internal dispatch failure.</td></tr></tbody></table>

### Example

#### Request

```javascript
const response = await fetch(RPC_URL, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: 1,
    method: "sendBatch",
    params: [
      [
        "4hXTCkRzt9WyecNzV1XPgCDfGAZzQKNxLXgynz5QDuWWPSAZBZSHptvWRL3BjCvzUXRdKvHL2b7yGrRQcWyaqsaBCncVG7BFggS8w9snUts67BSh3EqKpXLUm5UMHfD7ZBe9GhARjbNQMLJ1QD3Spr6oMTBU6EhdB4RD8CP2xUxr2u3d6fos36PD98XS6oX8TQjLpsMwncs5DAMiD4nNnR8NBfyghGCWvCVifVwvA8B8TJxE1aiyiv9eL5MKs6",
        "5jKVoAQRSqLzundGBFXmP3MqL5HEytHHijCN2sJFueRqgGQqJSLarjNDQRNJbG3ADFaDj1LE3z..."
      ],
      {
        "mevProtect": true
      }
    ]
  })
});
const result = await response.json();
console.log(result);
```

#### Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": [
    "5VERv8NMhKqATzGBGFst5oBZoJN8sW2azHjfYFWRnMC9bYg2nFQrpRNr3zetGhKRKXnR8L1m7fBHU9bVNFRFcxk2",
    "2xPkqLFhNk8iFhKt7MAqGumnaYSJdETxr5L4sKbqgjAeZR3ynk14Y6RCf4vUGDJ9dM2kT3qXMGJY9x5vNR1QWCW"
  ]
}

```

#### Code Example

{% tabs %}
{% tab title="TypeScript" %}
```typescript
interface SendBatchOptions {
  /** When `true`, enables MEV protection for all transactions in the batch. @default false */
  mevProtect?: boolean;
}

interface JsonRpcResponse<T> {
  jsonrpc: "2.0";
  id: number;
  result?: T;
  error?: { code: number; message: string };
}

async function sendBatch(
  rpcUrl: string,
  transactions: string[], // base64-encoded serialized Solana transactions
  options: SendBatchOptions = {}
): Promise<string[]> { // returns transaction signatures
  const params: [string[], SendBatchOptions?] = [transactions];
  if (Object.keys(options).length > 0) {
    params.push(options);
  }

  const response = await fetch(rpcUrl, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      jsonrpc: "2.0",
      id: 1,
      method: "sendBatch",
      params,
    }),
  });

  const data: JsonRpcResponse<string[]> = await response.json();

  if (data.error) {
    throw new Error(`sendBatch failed: ${data.error.message}`);
  }

  return data.result!;
}

// Usage
const signatures: string[] = await sendBatch(
  "https://your-rpc-endpoint.com",
  [serializedTx1, serializedTx2],
  { mevProtect: true }
);
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
async function sendBatch(rpcUrl, transactions, options = {}) {
  const params = [transactions];
  if (Object.keys(options).length > 0) {
    params.push(options);
  }
  const response = await fetch(rpcUrl, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      jsonrpc: "2.0",
      id: 1,
      method: "sendBatch",
      params,
    }),
  });
  const data = await response.json();
  if (data.error) {
    throw new Error(`sendBatch failed: ${data.error.message}`);
  }
  return data.result; // string[]
}
// Usage
const signatures = await sendBatch(
  "https://your-rpc-endpoint.com",
  [serializedTx1, serializedTx2],
  { mevProtect: true }
);
```
{% endtab %}
{% endtabs %}

### Notes

* All transactions in the batch must meet the minimum tip requirement.
* A duplicate signature within a batch returns an error. Transactions before the duplicate may already have been sent.
* Transactions are processed individually after submission; a successful response means the transactions were accepted, not confirmed on-chain.

***

**Need Help?**

Join our[ **Discord**](https://discord.gg/2UfWGtUDtN) for technical guides, integration support, and live updates.
