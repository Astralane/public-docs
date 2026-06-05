---
description: How to submit transaction
icon: paper-plane
---

# Submit Transactions

Astralane provides a multitude of options for your needs, here are some of RPC methods we offer for Transaction sending.&#x20;

***

## sendTransaction

{% hint style="info" %}
🚀 For the fastest transaction execution, we recommend using our `sendTransaction` method.
{% endhint %}

This RPC call is fully compatible with all Solana libraries, making it a simple drop-in replacement for your existing workflows. It routes through our partner **SWQoS** clients, including **Jito** and **Paladin** (higher min tip), and provides direct routing to leading **block builders** like **Harmonic** and **BAM**. Additionally, it supports integration with **custom schedulers** such as **Rakurai**, ensuring optimized transaction processing, reduced latency, and maximum reliability across the network.

Just insert the URL where you would place a RPC URL and send your txns how you send normally. The only change is to add an instruction to tip.

#### URI Params

<table><thead><tr><th width="171">Param</th><th width="120">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>api-key</code></td><td>String</td><td><strong>Mandatory</strong>, to set api key for authentication</td></tr><tr><td><code>mev-protect</code></td><td>Boolean</td><td><strong>Optional</strong>, To set mev protect, default is <code>false</code></td></tr><tr><td><code>swqos-only</code></td><td>Boolean</td><td><strong>Optional</strong>, if set <code>true</code> , txn will be only sent via swqos, default is <code>false</code></td></tr></tbody></table>

Example :&#x20;

```
https://fra.gateway.astralane.io/iris?api-key=APIKEY&mev-protect=true
```

#### JSON-RPC params

<table><thead><tr><th width="231">Parameter</th><th width="138">Type</th><th>Description</th></tr></thead><tbody><tr><td>Encoded Transaction</td><td>String</td><td>A base64 encoded transaction.</td></tr><tr><td>Transaction Configuration</td><td>JSON Object</td><td>Its recommended to put<br>- <code>encoding</code> as <code>base64</code><br>- <code>skipPreflight</code> as <code>true</code></td></tr><tr><td>MeV Protect</td><td>JSON Object</td><td>Optional, setting it true will enable mev protect, Default is false.</td></tr></tbody></table>

Example :&#x20;

```
"params" : [                     // params is an array
    <encoded_transaction>,
    <Transaction Configuration>,
    <mevProtect true/false>
]
```

Example Request JSON :&#x20;

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendTransaction",
  "params": [
    "base64_encoded_txn1",
    {
      "encoding": "base64",
      "skipPreflight": true
    },
    { "mevProtect": true }
  ]
}
```

Example Success Response JSON :

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": "Signature"
}
```

Example Error Response JSON :

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "error": {
        "code": -32600,
        "message": "Error message"
    }
}
```

{% tabs %}
{% tab title="Rust" %}
<pre class="language-rust"><code class="lang-rust"><strong>
</strong>const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 10000; // added for spam prevention

fn send_tx_tipped(
    ixs: &#x26;mut Vec&#x3C;Instruction>,
    signer: &#x26;Keypair,
    rpc_client: &#x26;RpcClient, // https://api.mainnet-beta.solana.com
    astralane_txn_sender: &#x26;RpcClient // Astralane RPC 
) {
    let tip_ix = system_instruction::transfer(&#x26;signer.pubkey(), &#x26;TIP, MIN_TIP_AMOUNT);
    ixs.push(tip_ix);

    let blockhash = rpc_client.get_latest_blockhash().unwrap();
    let tx = Transaction::new_signed_with_payer(ixs, Some(&#x26;signer.pubkey()), &#x26;[signer], blockhash);
    let encoded_tx = base64::prelude::BASE64_STANDARD.encode(&#x26;bincode::serialize(tx).unwrap());
    
    let response = client
        .post(url)
        .header("api_key", "xxx")
        .json(&#x26;json! ({
            "jsonrpc": "2.0",
<strong>            "id": 1,
</strong>            "method": "sendTransaction",
            "params": [
                encoded_tx, 
                {
                    "encoding": "base64",
                    "skipPreflight": true,
                },
                { "mevProtect": true }// Mev protect enable, Optional, Default is false
            ]
        }))
        .send()
        .await;
}
}
</code></pre>
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Please do note that the sendTransaction endpoint supports **max\_retries** feature which is useful for traders which don't want our staked nodes to retry their txns. Reach out to us for more info on ideal use cases.
{% endhint %}

***

## sendBundle

If your operation depends on atomic txn execution then use our sendBundle endpoint to send atomically executing transactions.

Send up to **4 transactions** in a single atomic bundle. These transactions are executed **sequentially** , ensuring precision and reliability. If any transaction fails, the entire bundle is reverted—guaranteeing consistency and eliminating partial failures.&#x20;

<table><thead><tr><th width="231">Parameter</th><th width="138">Type</th><th>Description</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td>Optional, If a bundle only has 1 txn with this parameter as <code>false</code>, then it will also sent via <code>sentTransaction</code> pipeline. Default is <code>false</code></td></tr></tbody></table>

{% tabs %}
{% tab title="Rust" %}
<pre class="language-rust"><code class="lang-rust">const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 10000; // added for spam prevention

<strong>async fn send_bundle(
</strong>    ixs: &#x26;mut Vec&#x3C;Instruction>,
    signer: &#x26;Keypair,
    client: reqwest::Client,
    blockhash: Hash,
    url: String,
) {
    let tip_ix = system_instruction::transfer(&#x26;signer.pubkey(), &#x26;TIP, MIN_TIP_AMOUNT);
    ixs.push(tip_ix);
    let tx = Transaction::new_signed_with_payer(ixs, Some(&#x26;signer.pubkey()), &#x26;[signer], blockhash);
    let encoded_tx = base64::prelude::BASE64_STANDARD.encode(&#x26;bincode::serialize(tx).unwrap());
    let response = client
        .post(url)
        .header("api_key", "xxx")
        .json(&#x26;json! ({
            "jsonrpc": "2.0",
            "id": 1,
            "method": "sendBundle",
            "params": [[encoded_tx]],
        }))
        .send()
        .await;
}
</code></pre>
{% endtab %}

{% tab title="Golang" %}
_Code example for sendIdeal endpoint usage in Go given below:_

```go
package main

import (
	"bytes"
	"context"
	"encoding/base64"
	"encoding/json"
	"fmt"
	"github.com/davecgh/go-spew/spew"
	"github.com/gagliardetto/solana-go/programs/system"
	"log"
	"net/http"
	"time"

	"github.com/gagliardetto/solana-go"
	"github.com/gagliardetto/solana-go/rpc"
)

func main() {
	payerPrivateKeyBase58 := "YOUR_WALLET_PRIVATE_KEY"
	astralaneTipAddressBase58 := "astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"
	rpcEndpoint := "DM_US_FOR_TRIAL_RPC_ENDPOINTS"
	astralaneEndpoint := "http://fr.gateway.astralane.io/iris?api-key=YOUR_ASTRALANE_API_KEY"

	signer, err := solana.PrivateKeyFromBase58(payerPrivateKeyBase58)
	if err != nil {
		log.Fatalf("Failed to load private key: %v", err)
	}

	client := rpc.New(rpcEndpoint)
	recentBlockhash, err := client.GetLatestBlockhash(context.Background(), rpc.CommitmentFinalized)
	if err != nil {
		log.Fatalf("Failed to get recent blockhash: %v", err)
	}

	astralaneTipAddress, err := solana.PublicKeyFromBase58(astralaneTipAddressBase58)
	if err != nil {
		log.Fatalf("Failed to decode astralane tip address: %v", err)
	}
	tx, err := solana.NewTransaction(
		[]solana.Instruction{
			system.NewTransferInstruction(
				200_000,
				signer.PublicKey(),
				astralaneTipAddress,
			).Build(),
		},
		recentBlockhash.Value.Blockhash,
		solana.TransactionPayer(signer.PublicKey()),
	)
	if err != nil {
		log.Fatalf("Failed to create transaction: %v", err)
	}

	if _, err := tx.Sign(func(key solana.PublicKey) *solana.PrivateKey {
		return &signer
	}); err != nil {
		log.Fatalf("Failed to sign transaction: %v", err)
	}

	spew.Dump(tx)

	out, err := tx.MarshalBinary()
	if err != nil {
		log.Fatalf("Marshaling to binary failed: %v", err)
	}
	encodedTx := base64.StdEncoding.EncodeToString(out)

	decoded, err := base64.StdEncoding.DecodeString(encodedTx)
	if err != nil {
		log.Fatalf("Sanity check decode failed: %v", err)
	}
	if !bytes.Equal(decoded, out) {
		log.Fatalf("Sanity check failed: decoded transaction does not match original binary")
	}

	payload := map[string]interface{}{
		"jsonrpc": "2.0",
		"id":      1,
		"method":  "sendBundle",
		"params": []interface{}{
			[]string{encodedTx},
		},
	}

	response, err := sendBundleRequest(astralaneEndpoint, payload)
	if err != nil {
		log.Fatalf("Bundle RPC failed: %v", err)
	}

	log.Printf("Bundle RPC response: %s", response)
}

func sendBundleRequest(endpoint string, payload map[string]interface{}) (string, error) {
	jsonData, err := json.Marshal(payload)
	if err != nil {
		return "", fmt.Errorf("Failed to marshal JSON: %w", err)
	}

	client := &http.Client{Timeout: 15 * time.Second}
	resp, err := client.Post(endpoint, "application/json", bytes.NewBuffer(jsonData))
	if err != nil {
		return "", fmt.Errorf("Request to Bundle RPC failed: %w", err)
	}
	defer resp.Body.Close()

	var result map[string]interface{}
	if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
		return "", fmt.Errorf("Failed to decode Bundle RPC response: %w", err)
	}

	responseBytes, _ := json.MarshalIndent(result, "", "  ")
	return string(responseBytes), nil
}
```

For further explanation please refer Rust docs on step by step details on whats happening.
{% endtab %}
{% endtabs %}

**Request:**

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendBundle",
  "params": [
    [
      "base64_encoded_txn1",
      "base64_encoded_txn2",
      "base64_encoded_txn3"
    ],
    {
      "encoding": "base64",
      "mevProtect": true,
      "revertProtection": false
    }
  ]
}
```

**Response:**

List of signatures for your transaction&#x20;

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        "37Dxw2nJYw3T8JVqenPQMf39VJ9CNZYCyQm67b6nRj6fa6UjQ1UuLqFvh3wJ2G7LcMuZn4oq5kDt2A2CEXfi8D8"
    ]
}
```

{% hint style="info" %}
Note: MAX\_TRANSACTIONS\_IN\_BUNDLE = 4
{% endhint %}

_For enhanced MEV Protection on bundles use any valid Solana public key starting with `jitodontfront` and `dontbund1e`. Txns containing these instruction will be rejected by Jito/Harmonic block engines unless its at index zero._&#x20;

_Read more about it here:_ [_Jito_](https://docs.jito.wtf/lowlatencytxnsend/#sandwich-mitigation)_,_ [_Harmonic_](https://docs.harmonic.gg/searchers/bundle-control-accounts)&#x20;

***

## sendIdeal

Perfect for snipers!  Due to the segregation in validators as JITO validators and normal ones, traders are often conflicted between spending more on jito tips vs more in priority fees. Durable nonces offer a way to mitigate this issue.

Our sendIdeal RPC method accepts two transactions:

* One with a **high priority fee + min tip**
* Another with a **high tip + lower priority fee**

We route them through our advanced **SWQoS and bundling pipelines**. Using **durable nonces**, once one transaction lands, the other is automatically canceled—ensuring optimal efficiency and cost savings.\
\
if you don't want to manage the durable nonce accounts on your own we also provide a managed service for this, We create a nonce account for each api-key you use and you can query them using our **getNonce** rpc call. Follow the below integration steps for best utilization of this feature:&#x20;



{% tabs %}
{% tab title="Rust" %}
#### **Step 1 : Generate Nonce instruction**

```rust
use solana_sdk::hash::Hash;
use solana_sdk::pubkey::Pubkey;
use solana_sdk::signature::{EncodableKey, Keypair, Signer};

async fn get_nonce(
    client: reqwest::Client,
    url: String,
    auth_key: String,
) {
    let response = client
        .post(url)
        .header("api_key", "xxx")
        .json(&json! ({
            "jsonrpc": "2.0",
            "id": 1,
            "method": "getNonce",
            "params": [api_key], // provided during onboarding
        }))
        .send()
        .await;
    let result = response["result"].clone();
    let nonce = result["nonce"].as_str().unwrap();
    let nonce_account = Pubkey::from_str(result["nonceAccount"].as_str().unwrap()).unwrap();
    let nonce_authority =
        Pubkey::from_str(result["nonceAuthority"].as_str().unwrap()).unwrap();
    let nonce_as_hash = Hash::from_str(nonce).unwrap();
}
```

{% hint style="info" %}
if you already have an existing nonce account, you can just pass in your nonce account, you can simply pass in your nonce account public key instead of the API key. The response will still include your parsed nonce.
{% endhint %}

To use this nonce, just include an **advance nonce instruction** as the first instruction in your transactions.

#### **Step 2 : Generate your txn and Partial Sign it before submission**

```rust
const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 10000; // added for spam prevention

async fn send_ideal(
    signer: &Keypair,
    client: reqwest::Client,
    nonce: Hash,
    instructions: Vec<Instruction>,
    nonce_authority: &Pubkey,
    nonce_account: &Pubkey,
) {
    // add advance nonce instruction
    let advance_nonce = solana_sdk::system_instruction::advance_nonce_account(nonce_account, nonce_authority);

    let low_tip_high_fee_ixs = vec![
        advance_nonce.clone(),
        solana_sdk::compute_budget::ComputeBudgetInstruction::set_compute_unit_price(
            10 * MICRO_LAMPORTS_PER_LAMPORTS,
        ),
        // add your instructions here
        solana_sdk::system_instruction::transfer(&signer.pubkey(), &TIP, MIN_TIP_AMOUNT),
    ];

    let high_tip_low_fee_ixs = vec![
        advance_nonce,
        solana_sdk::compute_budget::ComputeBudgetInstruction::set_compute_unit_price(
            100,
        ),
        // add your instructions here
        solana_sdk::system_instruction::transfer(&signer.pubkey(), &TIP, 100 * MIN_TIP_AMOUNT),
    ];


    //add high  transaction priority fee and min tip
    let mut low_tip_high_fee_tx = Transaction::new_with_payer(&low_tip_high_fee_ixs, Some(&signer.pubkey()));
    low_tip_high_fee_tx.partial_sign(&[&signer], nonce);

    let mut high_tip_low_fee_tx = Transaction::new_with_payer(&high_tip_low_fee_ixs, Some(&signer.pubkey()));
    high_tip_low_fee_tx.partial_sign(&[&signer], nonce);

    let low_tip_high_fee_tx_encoded = base64::prelude::BASE64_STANDARD.encode(&bincode::serialize(low_tip_high_fee_tx).unwrap());
    let high_tip_low_fee_tx_encoded = base64::prelude::BASE64_STANDARD.encode(&bincode::serialize(high_tip_low_fee_tx).unwrap());

    let response = client
        .post(url)
        .json(&json! ({
            "jsonrpc": "2.0",
            "id": 1,
            "method": "sendIdeal",
            "params": [[low_tip_high_fee_tx_encoded, high_tip_low_fee_tx_encoded]],
        }))
        .send()
        .await;
}
```

{% hint style="info" %}
When using Astralane's managed nonce accounts, ensure that you use the **partial\_sign** method to sign your transactions. If you are not utilizing managed nonce accounts, proceed with the standard sign method.
{% endhint %}
{% endtab %}

{% tab title="Typescript" %}
_Code example for sendIdeal endpoint usage in TypeScript given below:_

```javascript
import {
  Connection,
  Keypair,
  PublicKey,
  SystemProgram,
  Transaction,
  LAMPORTS_PER_SOL,
  ComputeBudgetProgram,
} from '@solana/web3.js';
import * as bs58 from 'bs58';


 async sendTransTest() {
    try {
      const result = await fetch(
        `http://fr.gateway.astralane.io/iris?api-key=<api-key>`,
        {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            jsonrpc: '2.0',
            id: 1,
            method: 'getNonce',
            params: [
              'api-key',
            ],
          }),
        },
      );
      const res = await result.json();
      const data = res as any;
      const nonceAccount = new PublicKey(data.result.nonceAccount);
      const nonceAuthority = new PublicKey(data.result.nonceAuthority);
      const nonce = data.result.nonce;
      const advanceNonce = SystemProgram.nonceAdvance({
        noncePubkey: nonceAccount,
        authorizedPubkey: nonceAuthority,
      });

      const secretKey = Keypair.fromSecretKey(
        bs58.default.decode(
          'wallet-private-key',
        ),
      );

      const wallet = secretKey;
      //Tip account
      const toPublicKey = new PublicKey(
        'astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm',
      );
      const MIN_TIP_AMOUNT = 10000;

      const lowTipHighFeeIx = [
        advanceNonce,
        ComputeBudgetProgram.setComputeUnitPrice({
          microLamports: 1 * 100000,
        }),
        SystemProgram.transfer({
          fromPubkey: wallet.publicKey,
          toPubkey: toPublicKey,
          lamports: MIN_TIP_AMOUNT,
        }),
      ];

      const HighTipLowFeeIx = [
        advanceNonce,
        ComputeBudgetProgram.setComputeUnitPrice({
          microLamports: 1 * 100,
        }),
        SystemProgram.transfer({
          fromPubkey: wallet.publicKey,
          toPubkey: toPublicKey,
          lamports: 100 * MIN_TIP_AMOUNT,
        }),
      ];

      const htlfTransaction = new Transaction().add(...HighTipLowFeeIx);
      htlfTransaction.recentBlockhash = nonce;
      htlfTransaction.feePayer = wallet.publicKey;
      htlfTransaction.partialSign(wallet);
      const highTipLowFeeEncoded = Buffer.from(
        htlfTransaction.serialize({
          requireAllSignatures: false,
          verifySignatures: false,
        }),
      ).toString('base64');

      const lthfTransaction = new Transaction().add(...lowTipHighFeeIx);
      lthfTransaction.recentBlockhash = nonce;
      lthfTransaction.feePayer = wallet.publicKey;
      lthfTransaction.partialSign(wallet);
      const lowTipHighFeeEncoded = Buffer.from(
        lthfTransaction.serialize({
          requireAllSignatures: false,
          verifySignatures: false,
        }),
      ).toString('base64');

      const final = await fetch(
        `http://fr.gateway.astralane.io/iris?api-key=<api-key>`,
        {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            jsonrpc: '2.0',
            id: 1,
            method: 'sendIdeal',
            params: [[highTipLowFeeEncoded, lowTipHighFeeEncoded]],
          }),
        },
      );
      const finalResult = await final.json();
      console.log(finalResult);

      return lowTipHighFeeEncoded;
    } catch (error) {
      console.log(error);
    }
  }
```

For further explanation please refer Rust docs on step by step details on whats happening.
{% endtab %}
{% endtabs %}

**Request:**

```jsx
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendIdeal",
  "params": [[
    "transction_with_large_tip_low_priority_fee",
    "transaction_with_large_priority_fee_low_tip"
  ]]
}
```

**Response:**

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        "<signature A>",
        "<signature B>"
    ]
}
```

The latency performance for your transactions will depend on current network dynamics. Kindly reach out to us on telegram for recommendations on ideal setup for your operation

***

## sendBatch

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

<table><thead><tr><th width="98">Code</th><th>Message</th><th>Description</th></tr></thead><tbody><tr><td><code>-32600</code></td><td><code>request size {n} exceeded max batch size {max}</code></td><td>Batch exceeds the maximum allowed size.</td></tr><tr><td><code>-32600</code></td><td><code>contains duplicate signature</code></td><td>Two or more transactions in the batch share the same signature.</td></tr><tr><td><code>-32600</code></td><td><code>Transaction sanitization errors</code></td><td>A transaction is malformed or does not meet the minimum tip requirement.</td></tr><tr><td><code>-32603</code></td><td><code>failed to send batch, internal error</code></td><td>Internal dispatch failure.</td></tr></tbody></table>

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
  transactions: string[], // base58-encoded serialized Solana transactions
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
* Duplicate signatures within a batch will cause the entire request to fail.
* Transactions are processed individually after submission; a successful response means the transactions were accepted, not confirmed on-chain.

***

**Need Help?**

Join our[ **Discord**](https://discord.gg/2UfWGtUDtN) for technical guides, integration support, and live updates.
