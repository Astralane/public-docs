---
description: Everything you need to get you up and running
icon: wrench
---

# Endpoints and Configs

## Endpoints

{% tabs %}
{% tab title="Iris" %}
**Transaction Sending Endpoints**:&#x20;

* Global Edge Endpoint : [https://edge.astralane.io/iris2?api-key=xxxx](https://edge.astralane.io/iris2?api-key=xxxx)
* Frankfurt (Recommended): [<mark style="color:blue;">http://fr.gateway.astralane.io/iris2?api-key=xxxx</mark>](https://fr.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 185.191.117.97
* Frankfurt: [<mark style="color:blue;">http://fr2.gateway.astralane.io/iris2?api-key=xxxx</mark>](https://fr2.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 45.139.132.160
* San Francisco: [http://la.gateway.astralane.io/iris2?api-key=xxxx](https://la.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 74.118.142.151
* Tokyo: [<mark style="color:blue;">http://jp.gateway.astralane.io/iris2?api-key=xxxx</mark>](http://jp.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 189.1.164.31
* New York: [<mark style="color:blue;">http://ny.gateway.astralane.io/iris2?api-key=xxxx</mark>](http://ny.gateway.astralane.io/iris?api-key=xxxx)
  * IPv4 : 64.130.45.19
* Amsterdam (Recommended): [http://ams.gateway.astralane.io/iris2?api-key=xxxx](http://ams.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 64.130.43.43
* Amsterdam 2: [http://ams2.gateway.astralane.io/iris2?api-key=xxxx](http://ams2.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 84.32.186.73 (Cherry Servers)
* Limburg: [http://lim.gateway.astralane.io/iris2?api-key=xxxx](http://lim.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 162.19.222.232
* Singapore: [http://sg.gateway.astralane.io/iris2?api-key=xxxx](http://sg.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 67.209.54.176&#x20;
* Lithuania: [http://lit.gateway.astralane.io/iris2?api-key=xxxx](http://lit.gateway.astralane.io/iris2?api-key=xxxx)
  * IPv4 : 84.32.97.47

{% hint style="info" %}
- Its recommended to broadcast your txns to both global edge endpoint as well as to the closest endpoint for optimal performance.
- You can also use HTTPS on above endpoints.
- Using an IPv4 address will be **faster** and will support QUIC.
{% endhint %}

**Available calls:**

* _sendTransaction_&#x20;
* _sendBundle_
* _sendIdeal_
* _getNonce_
* _sendPaladin_
* _getHealth_

**Tipping Address**:&#x20;

* astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF&#x20;
* astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm
* astra9xWY93QyfG6yM8zwsKsRodscjQ2uU2HKNL5prk
* astraRVUuTHjpwEVvNBeQEgwYx9w9CFyfxjYoobCZhL
* astraEJ2fEj8Xmy6KLG7B3VfbKfsHXhHrNdCQx7iGJK
* astraubkDw81n4LuutzSQ8uzHCv4BhPVhfvTcYv8SKC
* astraZW5GLFefxNPAatceHhYjfA1ciq9gvfEg2S47xk
* astrawVNP4xDBKT7rAdxrLYiTSTdqtUr63fSMduivXK

Recently Added:&#x20;

* AstrA1ejL4UeXC2SBP4cpeEmtcFPZVLxx3XGKXyCW6to
*  AsTra79FET4aCKWspPqeSFvjJNyp96SvAnrmyAxqg5b7
*  AstrABAu8CBTyuPXpV4eSCJ5fePEPnxN8NqBaPKQ9fHR
* AsTRADtvb6tTmrsqULQ9Wji9PigDMjhfEMza6zkynEvV
*  AsTRAEoyMofR3vUPpf9k68Gsfb6ymTZttEtsAbv8Bk4d
*  AStrAJv2RN2hKCHxwUMtqmSxgdcNZbihCwc1mCSnG83W
*  Astran35aiQUF57XZsmkWMtNCtXGLzs8upfiqXxth2bz
*  AStRAnpi6kFrKypragExgeRoJ1QnKH7pbSjLAKQVWUum
*  ASTRaoF93eYt73TYvwtsv6fMWHWbGmMUZfVZPo3CRU9C

#### **Min Tip and Rate Limits:**

{% content-ref url="../send-txn-fee-tiers.md" %}
[send-txn-fee-tiers.md](../send-txn-fee-tiers.md)
{% endcontent-ref %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Please create an account to our [user portal](https://portal.astralane.io/)  to get access to api keys for our transaction landing service and create a ticket in [our discord](https://discord.gg/2UfWGtUDtN) server.

During onboarding you will be provided an API key for using the above services and rate limits will be applied according to flexible pricing plans. For custom setups for colocation and hop optimization setups, create a ticket on our discord server.
{% endhint %}
