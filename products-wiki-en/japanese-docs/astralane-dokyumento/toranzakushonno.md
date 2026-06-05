---
description: 取引の提出方法
icon: paper-plane
---

# トランザクションの送信

Astralane はお客様のニーズに合わせてさまざまなオプションを提供しています。ここでは、トランザクションの送信に提供している RPC メソッドの一部を紹介します。

#### **sendTransaction**

この RPC 呼び出しは、すべての Solana ライブラリと完全に互換性があるため、既存のワークフローの簡単な代替品になります。Jito や Paladin (より高い最小チップ) などのパートナー SWQoS クライアントを介してルーティングされ、トランザクション処理の最適化と最大限の信頼性が確保されます。

RPC URL を挿入する場所に URL を挿入し、通常どおりにトランザクションを送信するだけです。唯一の変更点は、チップに指示を追加することです。

```
const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 100_000; // added for spam prevention

fn send_tx_tipped(
    ixs: &mut Vec<Instruction>, 
    signer: &Keypair, 
    rpc_client: &RpcClient
    ) {
    let tip_ix = system_instruction::transfer(&signer.pubkey(), &TIP, MIN_TIP_AMOUNT);
    ixs.push(tip_ix);

    let blockhash = rpc_client.get_latest_blockhash().unwrap();
    let tx = Transaction::new_signed_with_payer(ixs, Some(&signer.pubkey()), &[signer], blockhash);

    rpc_client.send_transaction(&tx).unwrap();
}
```



> sendTransaction エンドポイントは、max\_retries: None および min\_context\_slot: None 機能をサポートしていることにご注意ください。これは、ステークされたノードにトランザクションを再試行させたくないトレーダーにとって便利です。理想的な使用例の詳細については、当社にお問い合わせください。
>
> \*\*新機能 - 最小チップが Paladin への送信の最小チップ要件を満たしている場合、エンドポイントを使用して Paladin にブロードキャストすることもできます。利点の詳細については、以下をご覧ください。



#### sendBundle

操作がアトミック トランザクション実行に依存している場合は、sendBundle エンドポイントを使用してアトミックに実行されるトランザクションを送信します。

1 つのアトミック バンドルで最大 4 つのトランザクションを送信します。これらのトランザクションは順番に実行されるため、精度と信頼性が確保されます。いずれかのトランザクションが失敗すると、バンドル全体が元に戻されるため、一貫性が保証され、部分的な失敗が排除されます。



{% tabs %}
{% tab title="Rust" %}
```
const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 100_000; // added for spam prevention

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
{% endtab %}
{% endtabs %}



**Request:**

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendBundle",
  "params": [
    "base64_encoded_txn1",
    "base64_encoded_txn2"
    "base64_encoded_txn3"
    {
      "encoding": "base64",
      "mevProtect": true
    }
  ]
}
```



**Response:**

トランザクションの署名一覧

{% content-ref url="../../low-latency/submit-transactions/" %}
[submit-transactions](../../low-latency/submit-transactions/)
{% endcontent-ref %}

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        "37Dxw2nJYw3T8JVqenPQMf39VJ9CNZYCyQm67b6nRj6fa6UjQ1UuLqFvh3wJ2G7LcMuZn4oq5kDt2A2CEXfi8D8"
    ]
}
```

{% content-ref url="../../low-latency/submit-transactions/" %}
[submit-transactions](../../low-latency/submit-transactions/)
{% endcontent-ref %}

{% hint style="info" %}
注: MAX\_TRANSACTIONS\_IN\_BUNDLE = 4
{% endhint %}



#### sendIdeal

\
スナイパーに最適です! Jitoバリデータと通常のバリデータに分離されているため、トレーダーは Jito チップに多く費やすか、優先手数料に多く費やすかで葛藤することがよくあります。永続的な nonce は、この問題を軽減する方法を提供します。

当社の sendIdeal RPC メソッドは、2 つのトランザクションを受け入れます:

* 1 つは優先度の高い手数料 + 最小チップ
* もう 1 つは高いチップ + 優先度の低い手数料

これらのトランザクションは、高度な SWQoS およびバンドル パイプラインを介してルーティングされます。永続的な nonce を使用すると、1 つのトランザクションが到着すると、もう 1 つは自動的にキャンセルされるため、最適な効率とコスト削減が保証されます。

永続的な nonce アカウントを自分で管理したくない場合は、そのための管理サービスも提供しています。使用する API キーごとに nonce アカウントを作成し、getNonce rpc 呼び出しを使用してクエリできます。この機能を最大限に活用するには、以下の統合手順に従ってください。

{% tabs %}
{% tab title="Rust" %}
**ステップ 1:** Nonce命令を生成する

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
すでに nonce アカウントをお持ちの場合は、nonce アカウントを渡すだけで済みます。API キーの代わりに、nonce アカウントの公開キーを渡すだけです。応答には解析された nonce が含まれます。
{% endhint %}

この nonce を使用するには、トランザクションの最初の指示として事前 nonce 指示を含めるだけです。

**ステップ 2**: 送信前にトランザクションを生成し、部分的に署名する

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
Astralane の管理 nonce アカウントを使用する場合は、partial\_sign メソッドを使用してトランザクションに署名してください。管理 nonce アカウントを使用していない場合は、標準の署名メソッドを使用してください。\
\*\*新機能 - 最小チップが Paladin への送信の最小チップ要件を満たしている場合、エンドポイントを使用して Paladin にブロードキャストすることもできます。利点の詳細については、以下を参照してください。
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
      const MIN_TIP_AMOUNT = 100000;

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
トランザクションのレイテンシパフォーマンスは、現在のネットワークダイナミクスに依存します。操作に最適な設定に関する推奨事項については、Discord でお問い合わせください。
{% endhint %}

## **sendPaladin (Beta)**

Paladin は、トランザクションをリーダーに直接送信するためのより効率的な方法を提供するカスタム TPU ポート実装です。Paladin クライアントは現在、Solana ネットワークの 10% で実行されています (2025 年 3 月 10 日現在)。[Read more](https://send.paladin.one/)

**ステップ 1**: パラディン リーダーの追跡\
パラディン バリデーターはすべてのスロットに存在するわけではないため、パラディン リーダーがいるスロットを確認するために使用できるパラディン リーダー トラッカー エンドポイントを用意しました。リーダー情報に基づいてトランザクションを動的に送信するためのトラッカー統合に関する一般的なガイドラインを以下に示します。



1. 現在のエポックのすべての Palidator 公開鍵を取得します

```
⚔️ GET /api/palidators
```

```
[
    "Ss...Z77",
    "ACv...mi",
    "7Z...Z84",
]
```



2. 次のパリデーターリーダースロットを獲得

```
⚔️ GET /api/next_palidator
```

```
{
  "pubkey": "Csd...def",
  "leader_slot": 42424242,
  "context_slot": 42424242
}
```



3. 指定されたスロット以降に次のリーダー Palidator を獲得する

```
⚔️ GET /api/next_palidator/{slot}
```

```
{
  "pubkey": "Csd...def",
  "leader_slot": 42424242,
  "context_slot": 42424242
}
```

{% hint style="warning" %}
注意: 悪意のあるオペレーターの中には、Paladin を使用しているふりをする人もいます。そのため、このようなシナリオで悪用されないように、悪意のあるオペレーターに対するプロアクティブなブロックリストを実装することをお勧めします。これを実現する方法がわからない場合は、Discord でお問い合わせください。

Paladin バリデーターに送信されるトランザクションには、10 ランポート/CU という非常に小さな最小制限があります。
{% endhint %}



**ステップ 2: パラディン トランザクションの作成**\
send\_paladin メソッドを使用して、これらのスロット中にトランザクションを送信します。このエンドポイントでトランザクションを送信する際は、最小チップ要件と最小優先手数料要件を必ず遵守してください。

トランザクションの送信時には2つのモードがサポートされます `Fail on revert: True / False`

このエンドポイントに送信されたすべてのトランザクションは、小規模なオークションを経て、次の基準に基づいて順序付けされます。

1. 優先料金
2. タイブレーカーはAstralaneチップアドレスのチップによって決定されます。

落札した取引はパラディンポート経由で送信されます。 `failOnRevert: false`の場合、取引がオークションに勝てなかった場合は、SWQoS / Jito 経由で送信されます。

のために `failOnRevert true` 、これらのトランザクションは、パラディン リーダーのスロットの近くにない場合、Palidators にのみ送信されます。これらのトランザクションは破棄されます。

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendPaladin",
  "params": [
    "<base 64 tx>",
    {
      "failOnRevert": "false"
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
これを効率的に使用するには、Paladin Leader API エンドポイントを使用してアクティブなリーダーとスロットのタイミングを追跡し、それに応じてチップ/料金を調整してください。最適なコスト管理のために、専用のエンドポイントを介して構成を更新してください。
{% endhint %}

***

**助けが必要ですか?**

参加してください[ **Discord**](https://discord.gg/2UfWGtUDtN) 技術ガイド、統合サポート、ライブアップデートなど。
