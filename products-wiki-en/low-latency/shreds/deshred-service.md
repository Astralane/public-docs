---
description: Getting the Most Out of Automatic Transaction Decoding in Our Deshred Service
---

# Deshred service

## Deshred Service

#### What is the Deshred Service?

Raw shreds arrive out of order, fragmented, and erasure-coded - turning them into usable transactions requires FEC reconstruction, deshredding, and deserialization on every client.

The **Deshred Service** does this work for you. Instead of receiving raw UDP shreds and reconstructing entries yourself, you receive **already-decoded transactions**, streamed to you as soon as each slot's data is reconstructed.

**Why it matters**

Building your own deshred pipeline means handling:

* FEC set reconstruction (erasure coding recovery)
* Merkle proof validation
* Entry deserialization
* Duplicate/out-of-order shred handling

Getting this wrong costs latency - sometimes more than you gain from receiving raw shreds early in the first place. The Deshred Service centralizes this work on infrastructure tuned for it, so clients get decoded transactions at the same speed (or faster) than a naive in-house implementation, without maintaining the reconstruction logic themselves.

**Who this is for**

Deshred Service is designed for:

* MEV searchers who want transaction-level data without building shred parsing
* Market makers reacting to on-chain state changes
* Indexers and analytics platforms
* Teams already using Astralane Shreds who want a higher-level feed alongside or instead of raw shredsн

***

#### Pricing & Access

prices info HERE!!

***

#### FAQ

<details>

<summary>1. How is this different from raw Shreds?</summary>

Raw Shreds gives you UDP packets you reconstruct yourself. Deshred Service gives you already-decoded transactions, trading a small amount of latency for zero reconstruction overhead on your side.

</details>

<details>

<summary>2. Can I use Deshred Service and raw Shreds together?</summary>

\[ЗАПОЛНИТЬ]

</details>

<details>

<summary>3. Which regions are supported?</summary>

maybe need to describe

</details>
