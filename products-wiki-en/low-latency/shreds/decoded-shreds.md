---
description: Get rid of raw shreds decoding overhead
---

# Decoded Shreds

## Deshred Service

#### What is the Deshred Service?

Raw shreds arrive in a huge stream of raw encoded UDP packets - turning them into usable transactions requires reconstruction, decoding, filtration and ALT resoultion on every client.

Our **Decoded Shreds Service** does this work for you. Instead of receiving raw UDP shreds and doing the boilerplate work yourself, you receive **already decoded and filtered transactions**, streamed to you at the shreds speed.

**Why it matters**

Building your own deshred pipeline means handling:

* FEC set reconstruction (erasure coding recovery);
* Merkle proof validation;
* Entry deserialization;
* Duplicate/out-of-order shred handling;
* Filtering out unrelated transactions (vote, failed, unimportant programs);
* Resolving Address Lookup Table addresses.

Getting this wrong costs latency - sometimes more than you gain from receiving raw shreds early in the first place. The Deshred Service centralizes this work on infrastructure tuned for it, so clients get decoded transactions at the same speed (or faster) than a naive in-house implementation, without maintaining the reconstruction logic themselves.

**Who this is for**

Deshred Service is designed for:

* MEV searchers who want transaction-level data without building shred parsing;
* Market makers reacting to on-chain state changes;
* Indexers and analytics platforms;
* Teams already using Astralane Shreds who want a higher-level feed alongside or instead of raw shreds.

***

#### Get Access

To get access, you need to establish GRPC connection to one of our endpoints:

| Location             | URL                                   |
| -------------------- | ------------------------------------- |
| Fra (Terraswitch)    | http:://fr1.shreds.astralane.io:12000 |
| Fra (Cherry Servers) | http:://fr2.shreds.astralane.io:12000 |
| NY (Terraswitch)     | http:://ny1.shreds.astralane.io:12000 |

To authorise, provide `x-token` header with value of your shreds API key.

Proto file available here: [https://github.com/Astralane/shred-tools/blob/main/proto/geyser.proto#L9](https://github.com/Astralane/shred-tools/blob/main/proto/geyser.proto#L9)

#### Pricing

As a early beta, Decoded Shreds are available for all active shred users (tier-1 and tier-2) until 1st September. Prices are subject of change afterwards.&#x20;

***

#### FAQ

<details>

<summary>1. How is this different from raw Shreds?</summary>

Raw Shreds gives you UDP packets you reconstruct yourself. Deshred Service gives you already-decoded transactions, trading a small amount of latency for zero reconstruction overhead on your side.

</details>

<details>

<summary>2. Can I use Deshred Service and raw Shreds together?</summary>

Yes, as they have different use cases. For example, with raw Shreds you can feed your own RPC to get processed transactions faster.&#x20;

</details>

<details>

<summary>3. Which regions are supported?</summary>

Currently Fra and NY, rise a discord ticket if you want more.&#x20;

</details>
