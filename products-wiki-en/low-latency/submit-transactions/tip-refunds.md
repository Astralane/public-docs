---
icon: recycle
---

# Tip Refunds

Astralane Iris users are automatically eligible to receive SOL tip refunds, if the transactions sent to our middleware executes successfully on chain for a lower price. Tip refunds do not change how transactions are executed and users do not need to make any changes to be eligible for them.&#x20;

## What is a Tip rebate?

Due to validator client segmentation in Solana (primarily between **Jito,** **Agave and FD**), many users struggle to determine whether they should prioritize higher priority fees or higher tips to reliably land transactions.

With network competition at an all time high, users often overspend by sending unnecessarily high SOL tips. In many cases, these same transactions could have been executed just as quickly at a fraction of the SOL tip amount. Using Astralane’s validator sidecar network, users are now able to capture back some of these cost inefficiencies in the form of a Tip rebate.

Astralane’s system is designed to minimize overpayment. Every transaction sent through our infrastructure is routed via our internal SwQoS network, which improves landing probability and issues rebates for transactions that overpay relative to what was required to land that transaction onchain.&#x20;

## Which Transactions Receive Rebates

Rebates are applied to transactions that successfully land in a slot through Astralane’s swqos network. Eligibility depends on several dynamic factors that vary per block, including:

* How much network congestion and competition there was
* Whether Astralane's internal iris network made a profit on the txn and how much
* If the transaction was sent directly to Astralane Iris, or shared with other RPCs and landing services

{% hint style="info" %}
If you are an existing Astralane Iris user, or planning to use on your existing setup - kindly reach out to us on discord and we will enable it for you. Join our discord server here for support [https://discord.gg/2UfWGtUDtN](https://discord.gg/2UfWGtUDtN).&#x20;
{% endhint %}

## How to maximize both rebates and speed

Transactions which are sent directly to Astralane without using advanced nonce or not getting broadcasted by the user to other RPCs or landing services, will qualify for higher rebate tiers. This is because they maximize the probability of Astralane’s internal network successfully landing them, allowing us to distribute larger rebates.&#x20;

{% hint style="info" %}
Usually sending txns via our `sendTransaction` method results in faster execution and higher rewards, as it skips the auction time and lands through our internal swqos network.&#x20;

Other methods like `sendBundle` and `sendPaladin` and `sendIdeal` also quality for Tip rebates - however depends on the configs used for those methods. &#x20;

Since transaction sending is always latency sensitive, in order to improve inclusion in blocks, we recommend users to get a collocated server with us.
{% endhint %}

## Other optional privileges:

We offer different types of incentives based on transaction flow and tip volume, including:

* Discounted / Waiving off monthly fees
* Free access to higher TPS capacity
* Free access to our beta products for traders (gRPC, shreds, topOfBlock, etc)

## Tracking Rebates

Rebates are tracked based on your Iris API key provided by us. You can use the API key to fetch and check the rebates you are eligible for in our platform (currently in progress). And you will be able to withdraw the rebates from this at the end of every month.&#x20;

