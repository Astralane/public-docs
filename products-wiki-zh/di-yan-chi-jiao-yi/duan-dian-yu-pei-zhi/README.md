---
description: 快速上手，一站式搞定
icon: wrench
---

# 端点与配置

## Endpoints

{% tabs %}
{% tab title="Iris" %}
**交易发送端点**

*   法兰克福（推荐）：[http://fr.gateway.astralane.io/iris?api-key=xxxx](http://fr2.gateway.astralane.io/iris?api-key=xxxx)

    IPv4 :  185.191.117.97
* 法兰克福: [http://fr2.gateway.astralane.io/iris?api-key=xxxx](http://fr2.gateway.astralane.io/iris?api-key=xxxx)\
  IPv4：45.139.132.160
* 旧金山: [http://lax.gateway.astralane.io/iris?api-key=xxxx](http://lax.gateway.astralane.io/iris?api-key=xxxx)\
  IPv4：45.32.86.58
* 东京: [http://jp.gateway.astralane.io/iris?api-key=xxxx](http://jp.gateway.astralane.io/iris?api-key=xxxx)\
  IPv4：189.1.164.31
* 纽约: [http://ny.gateway.astralane.io/iris?api-key=xxxx](http://ny.gateway.astralane.io/iris?api-key=xxxx)\
  IPv4：64.130.45.19
* 阿姆斯特丹: [http://ams.gateway.astralane.io/iris?api-key=xxxx](http://ams.gateway.astralane.io/iris?api-key=xxxx)\
  IPv4：64.130.43.43
* 林堡: [http://lim.gateway.astralane.io/iris?api-key=xxxx](http://lim.gateway.astralane.io/iris?api-key=xxxx)\
  IPv4：162.19.222.232

{% hint style="info" %}
**以上地址均支持 HTTPS 访问。**
{% endhint %}

**Paladin 领导者追踪：**

**接口地址：**[http://paladin.astralane.io/api/palidators](http://paladin.astralane.io/api/palidators)

**可用方法：**

* _sendTransaction_
* _sendBundle_
* _sendIdeal_
* _getNonce_
* _sendPaladin_
* _getHealth_

_**小费地址（Tipping Address）：**_

* astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF&#x20;
* astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm
* astra9xWY93QyfG6yM8zwsKsRodscjQ2uU2HKNL5prk
* astraRVUuTHjpwEVvNBeQEgwYx9w9CFyfxjYoobCZhL
* astraEJ2fEj8Xmy6KLG7B3VfbKfsHXhHrNdCQx7iGJK
* astraubkDw81n4LuutzSQ8uzHCv4BhPVhfvTcYv8SKC
* astraZW5GLFefxNPAatceHhYjfA1ciq9gvfEg2S47xk
* astrawVNP4xDBKT7rAdxrLYiTSTdqtUr63fSMduivXK

#### 限流策略（Rate Limits）**:**

默认限流为 5 TPS（每秒 5 笔）适用于 `Iris 端点` 。\
需要更高 TPS 或“高频发送 + 自定义返利”方案？欢迎通过 Telegram 联系我们。

**鉴权方式（Authentication）：**

支持两种传递方式：

* 将 `api-key` 作为 URL 查询参数；
* 更推荐作为 HTTP Header 传递（隐私更佳），Header 名称为 `api-key`。

**最低小费（Min Tip）**

* 通过 `Iris endpoints` 发送的每笔交易，需包含一条向 Astralane 小费地址的系统转账指令，最低小费为 **0.00001 SOL**。
* 使用 `sendPaladin` 方法发送的交易，需包含同样的小费指令，最低 **0.0001 SOL**；同时计算单位价格（CU price）需≥ `100_000` micro lamports。
* 对于量化与高频用户，我们提供可协商的定制套餐，支持降低或免除最低小费要求。
{% endtab %}
{% endtabs %}



{% hint style="info" %}
接入与开通

* 请先填写 [Onboarding](https://forms.gle/G6NzFCYomgQ9HtVz6) 表单，并在 [Discord](https://discord.com/invite/2UfWGtUDtN) 服务器提交工单，申请交易落地服务的 API Key。
* Onboarding 完成后我们会发放 API Key，并依据灵活定价方案为你配置相应的频率与额度。
* 如需同机托管（colocation）或跳点优化（hop optimization）等深度定制部署，请在 Discord 创建工单，我们将为你一对一配置。
{% endhint %}
