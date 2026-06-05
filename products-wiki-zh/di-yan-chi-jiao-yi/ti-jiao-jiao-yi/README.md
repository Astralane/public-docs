---
description: 如何发送交易
icon: paper-plane
---

# 提交交易

Astralane 为不同场景提供多种选择。以下是我们用于发送交易的主要 RPC 方法。

***

## sendTransaction

{% hint style="info" %}
🚀 建议优先使用 `sendTransaction`，获取更快的落地速度。
{% endhint %}

* 与所有主流 Solana 库完全兼容，可直接替换你现有的 RPC 提交流程。
* 通过我们的 SWQoS 合作通道（包含 Jito 与 Paladin，后者要求更高的最小小费）进行路由，兼顾性能与可靠性。
* 用法与普通 RPC 完全一致：把 URL 换成我们的端点即可。唯一不同是需额外添加一条“小费”转账指令。

JSON-RPC 参数格式：

```
"params" : [                     // params 是一个数组
    <encoded_transaction>,
    <Transaction Configuration>,
    <mevProtect true/false>
]
```

<table><thead><tr><th width="231">参数</th><th width="138">类型</th><th>说明</th></tr></thead><tbody><tr><td>Encoded Transaction</td><td>String</td><td>使用 base64 编码的交易串。</td></tr><tr><td>Transaction Configuration</td><td>JSON Object</td><td>建议设置：<br>- <code>encoding</code> as <code>base64</code><br>- <code>skipPreflight</code> as <code>true</code></td></tr><tr><td>MeV Protect</td><td>JSON Object</td><td>可选。设为 true 将开启 MEV 保护，默认 false。</td></tr></tbody></table>

请求示例 JSON :&#x20;

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

{% tabs %}
{% tab title="Rust" %}
<pre class="language-rust"><code class="lang-rust">const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // 视接入区域选择对应小费地址
const MIN_TIP_AMOUNT: u64 = 10000; // 反垃圾的最小小费

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
                { "mevProtect": true }// 可选，默认 false
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
sendTransaction 支持 `max_retries` 参数。若你不希望我们质押节点为你的交易自动重试，可用该参数控制。欢迎联系我们了解适用场景。

**\*\*新增：若最小小费达到 Paladin 要求，该端点也可同时向 Paladin 广播（优势见后文）。**
{% endhint %}

***

## sendBundle

如果你的操作需要原子性落地，请使用 sendBundle。

* 单个原子包最多可包含 4 笔交易，按顺序依次执行。
* 任意一笔失败即整体回滚，确保一致性，避免“部分成功”。

<table><thead><tr><th width="231">参数</th><th width="138">类型</th><th>说明</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td>若一个 bundle 仅包含 1 笔交易且该值为 <code>false</code>，则会走 <code>sentTransaction</code> 路线。默认 <code>false</code>。</td></tr></tbody></table>

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

**请求示例：**

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

**响应示例：**

返回交易签名列表&#x20;

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
说明：MAX\_TRANSACTIONS\_IN\_BUNDLE = 4
{% endhint %}

***

## sendIdeal

**为狙击场景而生！**

由于验证者分化为 Jito 与非 Jito，两种付费路径（Jito tip vs 优先费）如何取舍常令人纠结。可利用可持久 nonce（durable nonce）来对冲不确定性。

sendIdeal 接受两笔交易：

* 交易 A：**高优先费 + 最低小费**
* 交易 B：**高小费 + 较低优先费**

我们会通过 **SWQoS 与打包管线并行路由**，并借助 **durable nonce** 确保一旦其中一笔上链，另一笔会自动作废，从而兼顾效率与成本。

如果你不想自行管理 nonce 账户，可使用我们的托管服务。我们会为你的每个 api-key 创建专属 nonce 账户，并可通过 **getNonce RPC** 查询。

建议按以下步骤集成：



{% tabs %}
{% tab title="Rust" %}
#### **Step 1 :** 获取 Nonce

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

已有自托管 nonce 账户？

可以直接在 params 传入你的 nonce 账户公钥，响应同样会返回解析后的 nonce。

使用时，注意把 advance nonce 指令放在交易的第一条指令。

#### **Step 2 :** 生成两笔交易并用 partial\_sign 预签名

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
    // 先添加 advance nonce 指令
    let advance_nonce = solana_sdk::system_instruction::advance_nonce_account(nonce_account, nonce_authority);

    let low_tip_high_fee_ixs = vec![
        advance_nonce.clone(),
        solana_sdk::compute_budget::ComputeBudgetInstruction::set_compute_unit_price(
            10 * MICRO_LAMPORTS_PER_LAMPORTS,
        ),
        // 你的任务指令
        solana_sdk::system_instruction::transfer(&signer.pubkey(), &TIP, MIN_TIP_AMOUNT),
    ];

    let high_tip_low_fee_ixs = vec![
        advance_nonce,
        solana_sdk::compute_budget::ComputeBudgetInstruction::set_compute_unit_price(
            100,
        ),
        // 你的任务指令
        solana_sdk::system_instruction::transfer(&signer.pubkey(), &TIP, 100 * MIN_TIP_AMOUNT),
    ];


    // 高优先费 + 最低小费
    let mut low_tip_high_fee_tx = Transaction::new_with_payer(&low_tip_high_fee_ixs, Some(&signer.pubkey()));
    low_tip_high_fee_tx.partial_sign(&[&signer], nonce);

    // 高小费 + 低优先费
    let mut high_tip_low_fee_tx = Transaction::new_with_payer(&high_tip_low_fee_ixs, Some(&signer.pubkey()));
    high_tip_low_fee_tx.partial_sign(&[&signer], nonce);

    // 编码
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
使用 Astralane 托管的 nonce 账户时，请务必使用 **partial\_sign**。若使用自管 nonce，可按常规 sign。

**\*\*新增：若最小小费满足 Paladin 要求，该端点也可同时向 Paladin 广播（优势见下文）。**
{% endhint %}
{% endtab %}

{% tab title="Typescript" %}
_下面是使用 TypeScript 调用 sendIdeal 端点的代码示例：_

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

你的交易时延表现会受当前网络状况影响。若需为你的业务获取更合适的参数与架构建议，欢迎在 Telegram 联系我们。
{% endtab %}
{% endtabs %}

**请求示例：**

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

**响应示例：**

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

实际时延表现受当前网络状态影响。如需为你的业务推荐更合适的参数与架构，请在 Telegram 联系我们。

***

## **sendPaladin (Beta)**

Paladin 是定制的 TPU 端口实现，可更高效地将交易直达当前 Leader。截止 2025-03-10，Paladin 客户端已覆盖约 10% 的 Solana 节点。[更多](https://send.paladin.one/)

**Step 1：追踪 Paladin 领导者**

由于 Paladin 验证者并非覆盖所有 slot，我们提供了“Paladin 领导者跟踪”服务，便于你按领导者动态路由交易。

> **强烈建议使用 gRPC 跟踪实时领导者（普通 RPC 时效性不足）。**
>
> 若暂时无法使用 gRPC，建议在配置中将 `enableFallback` 设为 true，以获得更稳的效果。



1. **获取当前 epoch 的 Palidator 公钥列表**

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

或参考： \`[https://api.paladin.one/validators](https://api.paladin.one/validators/)\`&#x20;



2. **获取下一位 Palidator 的领导槽位**

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



3. 查询某一槽位及之后的下一位 Palidator 领导

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
有极少数恶意参与者**可能伪装为 Paladin**。为防止被利用，**建议实现“恶意运营者”主动阻名单**。如不确定如何落地，请到我们的 [Discord](https://discord.gg/2UfWGtUDtN) 咨询。\
\
向 Paladin 验证者发送交易时，最小费用为 10 lamports/cu。
{% endhint %}



**Step 2：构造 Paladin 交易**

{% hint style="info" %}
请参考“**最低小费（Min Tip）**”的相关使用限制。
{% endhint %}

&#x20;Paladin slot 内通过 `sendPaladin`方法发送交易。务必满足最小小费与最小优先费的要求。

该端点的排序规则：

* 优先按 priority fee 排序；
* 若出现并列，再按向 Astralane 小费地址支付的小费高低决出先后。

<table><thead><tr><th width="231">参数</th><th width="134">类型</th><th>说明</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td>启用后，失败的交易将被丢弃，而不是以失败状态上链。默认 false。</td></tr><tr><td><pre><code>enableFallback
</code></pre></td><td>Boolean</td><td>设为 <code>true</code>：若当前 slot Leader 非 Paladin，或在 Paladin slot 的拍卖中未“胜出”，则自动回退到 sendTransaction 流水线继续发送。<br>设为 false：若在非 Paladin slot 提交，或竞拍未中标，将报错（如 <code>Invalid Request: transaction received at slot xxx which is not a paladin slot, fallback not enabled</code>），并被静默丢弃。</td></tr></tbody></table>

&#x20;

```json
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

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        "<signature A>",
    ]
}
```

{% hint style="info" %}
请结合 Paladin Leader API 持续追踪活跃领导者与 slot 时间窗口，按需调整小费/费用；并通过配置端点动态更新参数，**以获得更优的成本与成功率。**
{% endhint %}

***

**需要帮助？**

加入我们的 [**Discord**](https://discord.gg/2UfWGtUDtN) ，获取技术文档、接入支持与实时更新。
