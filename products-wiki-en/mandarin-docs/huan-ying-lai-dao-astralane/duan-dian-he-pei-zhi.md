---
description: 让您快速启动并运行所需的一切
---

# 🚅 端点和配置

### 端点和配置 — 启动和运行所需的一切

#### 🔗端点 Iris

交易发送端点：

* 法兰克福（推荐）：`http://fr.gateway.astralane.io/iris?api-key=xxxx`
  * IPv4 : 185.191.117.97
* 法兰克福: `http://fr2.gateway.astralane.io/iris?api-key=xxxx`
  * IPv4 : 45.139.132.160
* 东京: `http://jp.gateway.astralane.io/iris?api-key=xxxx`
  * IPv4 : 189.1.164.31
* 纽约:  `http://ny.gateway.astralane.io/iris?api-key=xxxx`
  * IPv4 : 64.130.45.19
* 旧金山:  `http://lax.gateway.astralane.io/iris?api-key=xxxx`&#x20;
  * IPv4 : 45.32.86.58
* 阿姆斯特丹: `http://ams.gateway.astralane.io/iris?api-key=xxxx`
  * IPv4 : 64.130.43.43
* 林堡省: `http://lim.gateway.astralane.io/iris?api-key=xxxx`
  * IPv4 : 162.19.222.232
* 立陶宛: `http://lit.gateway.astralane.io/iris?api-key=xxxx`
  * IPv4 : 84.32.97.47

{% hint style="info" %}
以上端点也可使用 HTTPS.
{% endhint %}

***

#### 🔭 Paladin 验证器领导追踪:

端点: [http://paladin.astralane.io/api/palidators](http://paladin.astralane.io/api/palidators)

可用通话：

* `sendTransaction`
* `sendBundle`
* `sendIdeal`
* `getNonce`
* `sendPaladin`
* `getHealth`

***

### 💸打赏地址：&#x20;

* astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF&#x20;
* astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm
* astra9xWY93QyfG6yM8zwsKsRodscjQ2uU2HKNL5prk
* astraRVUuTHjpwEVvNBeQEgwYx9w9CFyfxjYoobCZhL
* astraEJ2fEj8Xmy6KLG7B3VfbKfsHXhHrNdCQx7iGJK
* astraubkDw81n4LuutzSQ8uzHCv4BhPVhfvTcYv8SKC
* astraZW5GLFefxNPAatceHhYjfA1ciq9gvfEg2S47xk
* astrawVNP4xDBKT7rAdxrLYiTSTdqtUr63fSMduivXK



**速率限制（Rate Limits）**

默认情况下，`Iris` 端点的速率限制为 **5 TPS**（每秒事务数）。如需更高 TPS 或定制返利计划，欢迎通过 Telegram 联系我们。



**最低小费（Min Tip）**

* 发送到 `Iris` 端点的每笔交易都必须附带系统转账至 Astralane 打赏地址，最低小费为 **0.00001 SOL**。
* 使用 `sendPaladin` 方法发送的每笔交易，也必须附带系统转账至 Astralane 打赏地址，最低小费为 **0.0001 SOL**，且最低计算单元（compute unit price）为 **100,000 微 lamport（micro lamports）**。
* 对于高级交易发送者，我们还提供灵活方案，可在定制定价计划下享有更低甚至无最低小费。

***

{% hint style="info" %}
请填写我们的 [\[入职表单\]](https://docs.google.com/forms/d/e/1FAIpQLSdNOh2PvrV2VkU7MxXF6yVsFGcFa-bQbu4d-Q5Wh-vFzYehhQ/viewform) 以获取 API 密钥。同时，欢迎加入我们的 [\[Discord 服务器\]](https://discord.com/invite/2UfWGtUDtN) 以保持最新动态。

你可以通过查询参数或 HTTP 头（后者更隐私）使用 `api_key` 来附加 API 密钥。

在入职过程中，我们会为你提供 API 密钥，并根据灵活定价方案实施速率限制。若需定制化配置，例如邻近部署（colocation）或跳点优化（hop optimization），请在我们的 Discord 服务器上创建工单（ticket）。
{% endhint %}
