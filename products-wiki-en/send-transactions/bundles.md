---
description: Up to 4 transactions that land together or not at all
icon: layer-group
---

# Bundles

{% hint style="info" %}
This page covers bundles sent through Iris. To send bundles to the builders on Gravity Block Exchange, see [Submit Bundles](../gravity-block-exchange-gbx/submit-bundles.md). GBX bundles pay priority fees instead of tips.
{% endhint %}

If your operation depends on atomic txn execution then use our sendBundle endpoint to send atomically executing transactions.

Send up to **4 transactions** in a single atomic bundle. These transactions are executed **sequentially** , ensuring precision and reliability. If any transaction fails, the entire bundle is reverted—guaranteeing consistency and eliminating partial failures.&#x20;

#### Bundle Usage Guidelines

Please use `sendBundle` only when you need atomic execution of multiple transactions. Bundles are enabled per API key: see [Access and support](../start-here/access-and-support.md).

* We recommend maintaining a bundle success rate of around 15–20%, as processing unsuccessful bundles is costly for us to support. Bundle access may be temporarily disabled in cases of extremely low success rates if you haven’t received explicit approval from us to use bundles.
* We do not recommend using `sendBundle` for a single transaction. For single transactions, please use `sendTransaction` instead. If you use `sendBundle` for low-CU orders, keep a healthy landing rate and healthy Astralane tips. Reach out on Discord for recommended setups.

<table><thead><tr><th width="231">Parameter</th><th width="138">Type</th><th>Description</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td>Optional, If a bundle only has 1 txn with this parameter as <code>false</code>, then it will also be sent through the <code>sendTransaction</code> pipeline. Default is <code>false</code>. When enabled, transactions are not sent through the normal TPU pathway, only through private bundle pathways</td></tr></tbody></table>

{% tabs %}
{% tab title="Rust" %}
<pre class="language-rust"><code class="lang-rust">const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 1_000_000; // 0.001 SOL, the free-tier minimum

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
    let encoded_tx = base64::prelude::BASE64_STANDARD.encode(&#x26;bincode::serialize(&#x26;tx).unwrap());
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

{% tab title="Go" %}
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
				1_000_000, // 0.001 SOL, the free-tier minimum
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
