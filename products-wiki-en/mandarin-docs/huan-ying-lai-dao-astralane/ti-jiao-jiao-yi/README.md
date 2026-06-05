---
description: 如何提交交易
icon: paper-plane
---

# 提交交易

Astralane 为您的需求提供了多种选项，以下是我们为交易发送提供的一些 RPC 方法。

### sendTransaction

{% hint style="info" %}
若追求最快的交易执行速度，推荐使用 `sendTransaction` 方法。
{% endhint %}

此 RPC 调用兼容所有 Solana SDK/Libraries，可以无缝替换你已有的交易流程。它会通过 Astralane 的 SWQoS 合作路径（如 Jito 和 Paladin，需较高的小费），确保优化处理并提升执行可靠性。\
只需将 RPC URL 改为 Astralane 提供的，按照常规发送交易，并添加一条提示小费指令即可。

JSON-RPC 参数格式：

```
"params" : [                     // params is an array
    <encoded_transaction>,
    <Transaction Configuration>,
    <mevProtect true/false>
]
```

<table><thead><tr><th width="231">参数</th><th width="138">类型</th><th>类型</th></tr></thead><tbody><tr><td>Encoded Transaction</td><td>String</td><td>条经过 base64 编码的交易。</td></tr><tr><td>Transaction Configuration</td><td>JSON Object</td><td><p>建议设置以下参数：</p><ul><li><strong>encoding</strong> 设为 <code>base64</code></li><li><strong>skipPreflight</strong> 设为 <code>true</code></li></ul></td></tr><tr><td>MeV Protect</td><td>JSON Object</td><td>可选 将此参数设为 <code>true</code> 将启用 MEV 保护功能，默认为 <code>false</code>。</td></tr></tbody></table>

示例 JSON

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

```rust

const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 10000; // added for spam prevention

fn send_tx_tipped(
    ixs: &mut Vec<Instruction>,
    signer: &Keypair,
    rpc_client: &RpcClient, // https://api.mainnet-beta.solana.com
    astralane_txn_sender: &RpcClient // Astralane RPC 
) {
    let tip_ix = system_instruction::transfer(&signer.pubkey(), &TIP, MIN_TIP_AMOUNT);
    ixs.push(tip_ix);

    let blockhash = rpc_client.get_latest_blockhash().unwrap();
    let tx = Transaction::new_signed_with_payer(ixs, Some(&signer.pubkey()), &[signer], blockhash);
    let encoded_tx = base64::prelude::BASE64_STANDARD.encode(&bincode::serialize(tx).unwrap());
    
    let response = client
        .post(url)
        .header("api_key", "xxx")
        .json(&json! ({
            "jsonrpc": "2.0",
            "id": 1,
            "method": "sendTransaction",
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
}
```

> 请注意，sendTransaction 端点支持 max\_retries: None 和 min\_context\_slot: None 功能，这对于不希望我们的质押节点重试其交易的交易者非常有用。请联系我们以获取有关理想用例的更多信息。
>
> \*\*新 - 如果最小小费满足发送给 Paladin 的最小小费要求，端点也可用于向 Paladin 广播。阅读下面的更多优势。。



### sendBundle

\
当你的操作需要原子执行时，请使用 `sendBundle` 方法。\
此端点允许你一次性以原子方式发送至多 4 笔交易，它们会按顺序执行。如果其中任意一笔失败，整个 Bundle 会回滚，确保一致性与可靠性。

<table><thead><tr><th width="231">参数</th><th width="138">参数</th><th>说明</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td>可选 如果捆绑包中仅包含一笔交易且此参数为 <code>false</code>，则该交易也会通过 <code>sendTransaction</code> 管道发送。默认为 <code>false</code>。</td></tr></tbody></table>



{% tabs %}
{% tab title="Rust" %}
```
const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 10000; // added for spam prevention

async fn send_bundle(
    ixs: &mut Vec<Instruction>,
    signer: &Keypair,
    client: reqwest::Client,
    blockhash: Hash,
    url: String,
) {
    let tip_ix = system_instruction::transfer(&signer.pubkey(), &TIP, MIN_TIP_AMOUNT);
    ixs.push(tip_ix);
    let tx = Transaction::new_signed_with_payer(ixs, Some(&signer.pubkey()), &[signer], blockhash);
    let encoded_tx = base64::prelude::BASE64_STANDARD.encode(&bincode::serialize(tx).unwrap());
    let response = client
        .post(url)
        .header("api_key", "xxx")
        .json(&json! ({
            "jsonrpc": "2.0",
            "id": 1,
            "method": "sendBundle",
            "params": [[encoded_tx]],
        }))
        .send()
        .await;
}
```
{% endtab %}

{% tab title="golang" %}
```
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
				10000,
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

```
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

您的交易签名列表

```
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



### sendIdeal

非常适合狙击手！由于验证器分为 JITO 验证器和普通验证器，交易者经常在 jito 小费上花费更多和优先费用上花费更多之间产生矛盾。持久随机数提供了一种缓解此问题的方法。

我们的 sendIdeal RPC 方法接受两笔交易：

一笔交易具有高优先级费用 + 最低小费

另一笔交易具有高小费 + 低优先级费用

我们通过先进的 SWQoS 和捆绑管道路由它们。使用持久随机数，一旦一笔交易完成，另一笔交易就会自动取消 — 确保最佳效率和成本节约。

如果您不想自己管理持久随机数帐户，我们还为此提供托管服务，我们为您使用的每个 api 密钥创建一个随机数帐户，您可以使用我们的 getNonce rpc 调用查询它们。请按照以下集成步骤来充分利用此功能：



{% tabs %}
{% tab title="Rust" %}
**步骤 1**：生成Nonce指令

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
如果您已经有一个现成的 nonce 帐户，则只需传入您的 nonce 帐户，您只需传入您的 nonce 帐户公钥即可，而不必传入 API 密钥。响应仍将包含您解析的 nonce。
{% endhint %}

要使用这个 nonce，只需将提前 nonce 指令作为交易中的第一个指令即可。

**步骤 2**：生成交易并在提交前进行部分签名

```rust
const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 100_000; // added for spam prevention

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
使用 Astralane 的托管 nonce 账户时，请确保使用 partial\_sign 方法签署交易。如果您不使用托管 nonce 账户，请继续使用标准签名方法。

\*\*新 - 如果最小小费满足发送给 Paladin 的最小小费要求，则端点也可用于向 Paladin 广播。阅读下面的更多优势。
{% endhint %}
{% endtab %}

{% tab title="TypeScript" %}
TypeScript での sendIdeal エンドポイントの使用例を以下に示します。

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

さらに詳しい説明については、何が起こっているかを段階的に説明した Rust ドキュメントを参照してください。
{% endtab %}
{% endtabs %}



**Request**

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendIdeal",
  "params": [
    "transction_with_large_tip_low_priority_fee",
    "transaction_with_large_priority_fee_low_tip"
    {
      "encoding": "base64"
    }
  ]
}
```



**Response**

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        "<signature A>",
        "<signature B>"
    ]
}
```

{% hint style="info" %}
您的交易的延迟性能将取决于当前的网络动态。请通过电报联系我们，获取有关您操作的理想设置的建议
{% endhint %}

## **sendPaladin (Beta)**

Paladin 是一种定制的 TPU 端口实现，它提供了一种更高效的方式将您的交易直接发送给领导者。paladin 客户端目前在 10% 的 Solana 网络上运行（截至 2025 年 3 月 10 日）[Read more](https://send.paladin.one/)

**步骤 1：**&#x8DDF;踪圣骑士领袖\
由于并非所有插槽都有圣骑士验证器，因此我们提供了一个圣骑士领袖跟踪器端点，可用于了解哪些插槽有圣骑士领袖。以下是跟踪器集成的一些一般准则，可根据领袖信息动态发送交易：

> 必须使用 gRPC 跟踪当前领导者，因为普通 RPC 在此场景下速度过慢，无法有效使用。\
> 若当前无法使用 gRPC，建议将配置中的 `enableFallback` 设为 `true` 以获得最佳性能。



1. 获取当前时期的所有 Palidator 公钥

```
⚔️ GET http://paladin.astralane.io/api/palidators
```

```
[
    "Ss...Z77",
    "ACv...mi",
    "7Z...Z84",
]
```

或者，也可以使用：

```
https://api.paladin.one/validators
```

2. 获得下一任 Palidator 领导者职位

```
⚔️ GET http://paladin.astralane.io/api/next_palidator
```

```
{
  "pubkey": "Csd...def",
  "leader_slot": 42424242,
  "context_slot": 42424242
}
```



3. 在指定时间段内或之后获取下一任领导者 Palidator

```
⚔️ GET http://paladin.astralane.io/api/next_palidator/{slot}
```

```
{
  "pubkey": "Csd...def",
  "leader_slot": 42424242,
  "context_slot": 42424242
}
```

{% hint style="warning" %}
注意：某些恶意操作员有时会模仿自己使用圣骑士，因此为了防止在这些情况下被利用，我们建议您为恶意操作员实施主动阻止列表。如果您对如何实现这一点感到困惑，请通过我们的 [discord](https://discord.com/invite/2UfWGtUDtN) 与我们联系。

发送给圣骑士验证器的 txns 的最小限制为 10 lampors/cu。
{% endhint %}

第二步：构建 Paladin 交易

{% hint style="info" %}
请参考[最低小费要求](https://astralane.gitbook.io/docs/low-latency/endpoints-and-configs#min-tip)：使用限制
{% endhint %}

使用 `sendPaladin` 方法来针对这些 Paladin 领导者的 slots 发送交易。请确保在发送时满足 **最小手续费（min tip）** 和 **最小优先费（min priority fee）** 的要求。

交易将进入一个小型竞拍（auction）机制，最终排序原则如下：

1. 优先根据 **priority fees（优先费）** 排序；
2. 若出现优先费平局，则根据 **向 Astralane**[ **提示小费地址支付的小费数额（tip**](https://astralane.gitbook.io/docs/low-latency/endpoints-and-configs#min-tip)**）** 决定顺序&#x20;

**请求参数结构示例：**

<table><thead><tr><th width="231">Parameter</th><th width="134">Type</th><th>Description</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td><strong>必填</strong>。启用后，失败交易将被直接丢弃而非以失败状态上链。默认为 <code>false</code>。</td></tr><tr><td><pre><code>enableFallback
</code></pre></td><td>Boolean</td><td><strong>必填</strong>。设为 <code>true</code> 时：若当前时段领导者非 Paladin 或您的交易未能在 Paladin 领导者时段拍卖中胜出，交易将通过我们的 <code>sendTransaction</code> 管道发送；设为 <code>false</code> 时，将抛出错误提示（如“无效请求：交易在非 Paladin 时段接收且未启用降级机制，交易将被静默丢弃”）。</td></tr></tbody></table>

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendPaladin",
  "params": [
    "<base 64 tx>",
    {
      "revertProtection": false,
      "enableFallback" : true
    }
  ]
}
```

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        "<signature A>",
    ]
}
```

{% hint style="info" %}
为了有效使用此功能，请务必使用我们的 Paladin Leader API 端点来跟踪活跃领导者和时段时间，并相应地调整小费/费用。通过专用端点更新您的配置，以实现最佳成本控制。
{% endhint %}

***

**需要帮助吗？**

加入我们的[ **Discord**](https://discord.gg/2UfWGtUDtN) 获取技术指南、集成支持和实时更新。
