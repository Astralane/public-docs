# 🚅 エンドポイントと設定&#x20;

### 🚅 エンドポイントと設定 — 稼働に必要なすべてがここに



#### 🔗 **エンドポイント: Iris（アイリス）**



トランザクション送信用エンドポイント：

* フランクフルト: `http://fr.gateway.astralane.io/iris?api-key=xxxx`
* 東京: `http://jp.gateway.astralane.io/iris?api-key=xxxx`
* ニューヨーク: `http://ny.gateway.astralane.io/iris?api-key=xxxx`

***

#### 🔭 **Paladin（パラディン）リーダートラッキング**

エンドポイント: `http://fr.gateway.astralane.io/api/paladin`

**利用可能なAPIコール:**

* `sendTransaction`
* `sendBundle`
* `sendIdeal`
* `getNonce`
* `sendPaladin`

***

### 💸 チップ用アドレス`astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF` `astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm` `astra9xWY93QyfG6yM8zwsKsRodscjQ2uU2HKNL5prk` `astraRVUuTHjpwEVvNBeQEgwYx9w9CFyfxjYoobCZhL`

***

### 📊 レート制限

| エンドポイント | デフォルトTPS |
| ------- | -------- |
| Iris    | 5 TPS    |
| Paladin | 1 TPS    |

### 💰 最小チップ額

* **Iris エンドポイント**\
  各トランザクションに最低 `0.0001 SOL` のチップを設定し、\
  チップアドレスへのシステム送金命令を含める必要があります。
* **Paladin エンドポイント**\
  各トランザクションに最低 `0.00XX SOL` のチップを設定する必要があります。

✅ 高度なトランザクション送信者向けには、**カスタム価格設定プラン**により、\
最小チップ額の引き下げや免除が可能な柔軟なプランも提供しています。

> オンボーディングの際に、上記のサービスを利用するための[APIキー](https://forms.gle/G6NzFCYomgQ9HtVz6)発行されます。レート制限は柔軟な料金プランに応じて適用されます。コロケーションやホップ最適化セットアップなどのカスタム構成をご希望の場合は、[Discordサーバー](https://discord.com/invite/2UfWGtUDtN)でチケットを作成してください。

