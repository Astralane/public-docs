---
description: >-
  Как отправить транзакции  Astralane предоставляет множество вариантов под
  разные потребности. Ниже перечислены основные RPC-методы для отправки
  транзакций.
icon: paper-plane
---

# Отправить транзакцию

***

sendTransaction

> Для максимально быстрого исполнения транзакций рекомендуем использовать метод `sendTransaction`.

Этот RPC-вызов полностью совместим со всеми библиотеками Solana, что позволяет без труда заменить текущие процессы отправки транзакций. Запросы проходят через наших партнеров по SWQoS, включая Jito и Paladin (требует более высокий минимальный tip), что гарантирует оптимизированную обработку и максимальную надёжность.\
Просто используйте предоставленный URL вместо вашего обычного RPC-URL и отправляйте транзакции привычным способом — единственное отличие — нужно добавить инструкцию о tip.

Формат параметров JSON-RPC:

```
"params" : [                     // params is an array
    <encoded_transaction>,
    <Transaction Configuration>,
    <mevProtect true/false>
]
```

Параметры:

<table><thead><tr><th width="231">Параметр</th><th width="138">Тип</th><th>Тип</th></tr></thead><tbody><tr><td>Encoded Transaction</td><td>String</td><td>A base64 encoded transaction.</td></tr><tr><td>Transaction Configuration</td><td>JSON Object</td><td>Its recommended to put<br>- <code>encoding</code> as <code>base64</code><br>- <code>skipPreflight</code> as <code>true</code></td></tr><tr><td>MeV Protect</td><td>JSON Object</td><td>Optional, setting it true will enable mev protect, Default is false.</td></tr></tbody></table>

**Пример JSON:**

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
<pre class="language-rust"><code class="lang-rust"><strong>
</strong>const TIP: Pubkey = pubkey!("astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm"); // Use tip wallet depending on region of access
const MIN_TIP_AMOUNT: u64 = 100_000; // added for spam prevention

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

_(здесь добавляется инструкция передачи tip, затем транзакция отправляется через Astralane RPC)_

Важно отметить, что endpoint `sendTransaction` поддерживает параметр `max_retries`, полезный для трейдеров, которые не хотят, чтобы узлы автоматически повторно отправляли их транзакции. Для получения подробностей обратитесь к нам.\
Также — **новинка**: при соблюдении минимального tip этот endpoint позволяет отправлять транзакции через Paladin. Подробнее об этом ниже.

***

sendBundle

Если вам нужно атомарное выполнение операций, используйте endpoint `sendBundle`.

Вы можете отправлять до 4 транзакций в одном атомарном пакете. Они выполняются последовательно, и если хотя бы одна из транзакций завершается неудачей, весь пакет отменяется — это обеспечивает консистентность и исключает частичные ошибки.

<table><thead><tr><th width="231">Parameter</th><th width="138">Type</th><th>Description</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td>Optional, If a bundle only has 1 txn with this parameter as <code>false</code>, then it will also sent via <code>sentTransaction</code> pipeline. Default is <code>false</code></td></tr></tbody></table>

```rust
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

запрос

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

Пример ответа:

```
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": [
    "37Dxw2nJYw3T8JVqenPQMf39VJ9CNZYCyQm67b6nRj6fa6UjQ1UuLqFvh3wJ2G7LcMuZn4oq5kDt2A2CEXfi8D8"
  ]
}

```

> MAX\_TRANSACTIONS\_IN\_BUNDLE = 4

sendIdeal

Идеально для снайперов! Из-за разделения валидаций на JITO и обычные валидаторы, трейдеры часто колеблются между более высокими tip и higher priority fees. Durable nonces помогают смягчить эту дилемму.

Метод `sendIdeal` принимает две транзакции:

* одна с высоким priority fee и минимальным tip,
* другая — с высоким tip и низким priority fee.

Мы направляем их через продвинутые SWQoS-конвейеры и bundling. Используя durable nonce, при попадании одной транзакции — вторая автоматически отменяется, что обеспечивает эффективность и экономию. Если вы не хотите сами управлять nonce-аккаунтами, мы предлагаем управляемую службу: создаём nonce-аккаунт для каждого вашего API-ключа, доступен метод `getNonce` для получения nonce.

```rust
1. Получаем nonce:

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

2. Строим две транзакции с advance nonce инструкцией, tip и priority fee, подписываем частично, кодируем и отправляем:

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

Пример ответа:

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

Также **новинка**: при соблюдении минимального tip можно отправлять транзакции через Paladin — подробнее дальше.

sendPaladin (Beta)

**Paladin** — это собственная реализация TPU-портов, которая позволяет эффективнее отправлять транзакции напрямую лидерам слотов. Клиент Paladin сейчас работает примерно на **10 % сети Solana** (по состоянию на 10 марта 2025 года).&#x20;

**Шаг 1: Отслеживание лидеров Paladin**

Поскольку валидаторы Paladin не присутствуют во всех слотах, мы предоставляем endpoint-трекер лидеров Paladin для динамической отправки транзакций:

1. Получите список всех публичных ключей валидаторов–Paladitor’ов в текущем epoch:

```
GET http://paladin.astralane.io/api/palidators
```

2. Запросите следующий слот с Paladin лидером:

```
GET http://paladin.astralane.io/api/next_palidator
```

3. Или следующий лидер Paladin с указанного или более позднего слота:

```
GET http://paladin.astralane.io/api/next_palidator/{slot}
```

**Важно**: некоторые злоумышленники могут выдавать себя за Paladin-валидаторов — рекомендуется внедрять проактивные чёрные списки. Также установлена минимальная плата: **10 lamports/compute unit**.

**Шаг 2: Создание и отправка транзакции через Paladin**

Используйте метод `sendPaladin`, следя за выполнением минимальных требований по tip и priority fee. Все транзакции проходят небольшую аукционную сортировку по:

1. Priority fees,
2. В случае равенства — по величине tip, отправленного на адрес Astralane.

Пример запроса:

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "sendPaladin",
  "params": [
    "<base64_tx>",
    {
      "revertProtection": false,
      "enableFallback": false
    }
  ]
}

```

Возвращаемый ответ:

```
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": ["<signature A>"]
}
```

<table><thead><tr><th width="231">Параметр</th><th width="134">Тип</th><th>Описание</th></tr></thead><tbody><tr><td><pre><code>revertProtection
</code></pre></td><td>Boolean</td><td><strong>Обязателен</strong>. Если <code>true</code>, неудачная транзакция будет <em>отброшена</em>, а не засчитана как <em>failed</em>. По умолчанию — <code>false</code>.</td></tr><tr><td><pre><code>enableFallback
</code></pre></td><td>Boolean</td><td><strong>Обязателен</strong>. Если <code>true</code> и транзакция не выигрывает аукцион или слот-лидер не поддерживает Paladin, она перенаправляется через <code>sendTransaction</code>.</td></tr></tbody></table>

Для оптимального использования следите за активными лидерами с помощью API-трекера и корректируйте ваши fee/tip-настройки для экономии и эффективности.
