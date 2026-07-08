---
icon: shuttle-space
---

# Shreds

### What are Shreds?

On Solana, blocks are never transmitted as a complete unit.

Instead, the leader **streams the block as it is being produced**, splitting it into small packets called **shreds**. These are broadcast over UDP to the network and reassembled by validators in real time.

#### Why it matters

Latency on Solana is not a single event—it splits into two distinct phases:

1. **Transaction → Leader** (getting included)
2. **Leader → Network (via shreds)** (state becoming observable)

Most systems focus on phase (1).

But for latency-sensitive strategies, phase (2) is where edge is gained or lost. Shreds propagate over a **fanout-based UDP network (Turbine)**, meaning:

* Packets take different paths
* Arrival order is not guaranteed
* Timing varies across peers

As a result:

* Validators reconstruct blocks at **different wall-clock times**
* Earlier reconstruction = earlier access to:
  * account state
  * program outputs
  * orderbook changes

Even a few milliseconds here is material.

#### Who this is for

Astralane Shred Delivery is designed for:

* MEV searchers
* Market makers
* High-frequency traders
* Validator operators
* Infrastructure providers

If your strategy depends on reacting to state changes as early as possible, shred delivery is a must.

#### **How Astralane Shreds work**

Astralane provides **raw UDP shred streams** optimized for:

* Low-latency propagation
* Consistent shred arrival
* Efficient duplicate handling

Shreds are delivered directly over UDP to minimize overhead and avoid additional transport latency.

This allows clients to:

* Receive shreds earlier in the propagation cycle
* Reconstruct state faster
* Reduce dependency on slower or congested peers

### Pricing & Tips Setup

#### Shreds pricing

Instead of hard monthly commitments, we provide a pay-as-you-use model. If you accumulate enough **registered** tips for the last 24h, you will be automatically eligible to enter one of the shred tiers.

| Shreds Tier | Required 24h Tips | Description                                                              |
| ----------- | ----------------- | ------------------------------------------------------------------------ |
| tier-1      | 0.2 SOL           | Good for the dev testing, faster than Jito shreds.                       |
| tier-2      | 0.5 SOL           | Ideal for the ultra-low-latency shred delivery and production workloads. |

{% hint style="info" icon="info" %}
**Tip placement is flexible**: if you'd rather not add a tip to your entry transaction, you can attach it to the **closing leg** instead and register that transaction through ShredPay.
{% endhint %}

**What the tiers mean**

Your tier controls how many&#x20;validator-direct shred sources you receive. A higher tier covers more leaders,&#x20;which lowers your effective shred latency — you see more of the network's blocks&#x20;earlier.&#x20;

* **Tier-1** gives baseline coverage, enough for development and testing;
* **Tier-2** unlocks broader leader coverage for ultra-low-latency production  \
  workloads where every millisecond counts.

You can check your current tier and 24h tips at [https://portal.astralane.io/shreds](https://portal.astralane.io/shreds)

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

#### Grace period & downgrades

As soon as you cross a tier's 24h tip threshold, you are upgraded immediately.\
If your trailing-24h tips drop below your current tier, you are not dropped instantly. You keep the tier for a 24h grace window, then fall back one tier down.\
\
Downgrades step down one tier at a time (tier-2 → tier-1), never straight to no-access.

{% hint style="info" icon="info" %}
The grace period means a single quiet trading day won't cut off your shred feed
{% endhint %}

#### How your tips get counted

ShredPay is a separate HTTP service used only for registering tips - not for landing transactions.\
You send:

* signed transaction
* slot that triggered the transaction

**Request format:**

`POST` to your endpoint URL, JSON body (array):

```http
...
POST /shred-pay?api-key=<your-api-key> HTTP/1.1
Host: edge.astralane.io
Content-Type: application/json

[{ "transaction": "<base64 bincode-encoded signed tx>", "slot": 123456789 }]
...
```

**Response format:**

```json
{ "accepted": ["<your transaction signature>"] }
```

You can land your transactions anywhere — Iris or any other provider. That part is entirely up to you.\


{% hint style="info" icon="info" %}
**Tier counting**: a tip only counts once a copy of that transaction is sent to **ShredPay** over HTTP. Landing it somewhere doesn't register it — not even through Astralane's own services.
{% endhint %}

**No double-paying.** You don't pay Astralane twice. If you're already tipping \~0.5 SOL or more via Iris, that stays exactly the same — you just also send a **copy** of those tipped transactions to the ShredPay endpoint.

{% hint style="warning" %}
**Important**: starting **July 15**, tips sent through Iris without a separate ShredPay registration will no longer count toward your tier.
{% endhint %}

#### How to set up tipping&#x20;

The best use of such model is integrating tips into your searcher code:

```
// main searcher logic
...
if astralane_shred_was_a_source_signal() {
    land_transaction()              // any provider
    send_tx_copy_to_shredpay()     // registers the tip
}
```

**Full working example**: builds a tip transaction, lands it via Iris, and registers a copy through the ShredPay HTTP endpoint.

**View the example on** [**GitHub**](https://github.com/Astralane/shred-tools/tree/main/shreds-example)**.**

#### Tipping Address

* ASTZHptaMgYVMX6DAocDr1vVXLran5PpfKfQtVTSWkfE
* AStZiY6EE532nQBBogmvcWemc2bwg2kHuR4Jrd5Cqaq5
* AStzprSDj14m1pdMuuomjLQwUL5X4d4yEKwiFfzfUSeC
* ASTzWqJ1irGmrWZAkc1UUfVYbJknsxh7R3Jo4jYZ7Mbd

#### ShredPay endpoints

Send your [POST request](shreds.md#how-your-tips-get-counted) to the endpoint closest to your infra:

<table><thead><tr><th width="272.22222900390625">Region</th><th>HTTP Endpoint</th></tr></thead><tbody><tr><td>Global Edge Endpoint</td><td><a href="https://edge.astralane.io/shred-pay?api-key=xxxx">https://edge.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Frankfurt (Recommended)</td><td><a href="http://fr.gateway.astralane.io/shred-pay?api-key=xxxx">http://fr.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Frankfurt</td><td><a href="http://fr2.gateway.astralane.io/shred-pay?api-key=xxxx">http://fr2.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>San Francisco</td><td><a href="http://la.gateway.astralane.io/shred-pay?api-key=xxxx">http://la.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Tokyo</td><td><a href="http://jp.gateway.astralane.io/shred-pay?api-key=xxxx">http://jp.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>New York</td><td><a href="http://ny.gateway.astralane.io/shred-pay?api-key=xxxx">http://ny.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Amsterdam (Recommended)</td><td><a href="http://ams.gateway.astralane.io/shred-pay?api-key=xxxx">http://ams.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Amsterdam 2</td><td><a href="http://ams2.gateway.astralane.io/shred-pay?api-key=xxxx">http://ams2.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Limburg</td><td><a href="http://lim.gateway.astralane.io/shred-pay?api-key=xxxx">http://lim.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Singapore</td><td><a href="http://sg.gateway.astralane.io/shred-pay?api-key=xxxx">http://sg.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr><tr><td>Lithuania</td><td><a href="http://lit.gateway.astralane.io/shred-pay?api-key=xxxx">http://lit.gateway.astralane.io/shred-pay?api-key=xxxx</a></td></tr></tbody></table>

{% hint style="info" icon="info" %}
Get your API key from the Astralane portal. Requests without a valid key are rejected.
{% endhint %}

### Get Access to the Shreds

You can connect your shred stream directly through the [Astralane](https://astralane.io/) portal. If you run into any issues, open a support ticket in [Discord](https://discord.gg/2UfWGtUDtN), providing your desired region and destination `ip:port`.

Currently supported regions:&#x20;

* FRA (shred relay is on Fra2 Terraswitch and Cherry for co-location)
* NY (shred relay is on Ewr2 Terraswitch for co-location)

### Examples & Code

#### Libraries Required

<table><thead><tr><th width="170.01953125">Crate</th><th width="120.74609375">Version</th><th>Why</th></tr></thead><tbody><tr><td><code>solana-ledger</code></td><td><code>3.1.12</code></td><td>Provides <code>Shred</code> type, parsing, and <code>new_from_serialized_shred()</code>. Must enable the <code>agave-unstable-api</code> feature since the shred module is gated behind it in v3.</td></tr><tr><td><code>solana-packet</code></td><td><code>3.x</code></td><td>Provides <code>PACKET_DATA_SIZE</code> (1232 bytes) — the max size of a Solana UDP packet. Use this as your receive buffer size.</td></tr><tr><td><code>solana-clock</code></td><td><code>3.x</code></td><td>Provides the <code>Slot</code> type alias (<code>u64</code>).</td></tr></tbody></table>

#### Optional

<table><thead><tr><th width="180.75390625">Crate</th><th>Why</th></tr></thead><tbody><tr><td><code>solana-entry</code></td><td>Needed if you want to deserialize deshredded data into <code>Entry</code> objects (contains transactions).</td></tr><tr><td><code>bincode</code></td><td>Needed to deserialize the raw deshredded bytes into <code>Vec&#x3C;Entry></code>.</td></tr><tr><td><code>solana-streamer</code></td><td>High-performance packet receiver using <code>recvmmsg</code>. Useful if you need multi-threaded reception at scale, but overkill for basic usage.</td></tr><tr><td><code>solana-perf</code></td><td>Provides <code>Deduper</code> (bloom filter) for deduplicating shreds. Only needed at scale.</td></tr></tbody></table>

#### Cargo.toml

```toml
[dependencies]
solana-ledger = { version = "=3.1.12", features = ["agave-unstable-api"] }
solana-packet = "3"
solana-clock = "3"
```

**Note on GCC 15+**: `solana-ledger` pulls in RocksDB which fails to compile on GCC 15 due to missing `<cstdint>` includes. Add this workaround:

```toml
# .cargo/config.toml
[env]
CXXFLAGS = "-include cstdint"
```

#### Shred Wire Format

Every shred starts with a fixed common header at these byte offsets:

```
Offset  Size  Field
------  ----  -----
0       64    Signature (ed25519)
64       1    ShredVariant (identifies type: legacy data/code, merkle data/code)
65       8    Slot (u64 LE) — which slot this shred belongs to
73       4    Index (u32 LE) — position within the slot
77       2    Version (u16 LE) — shred version
79       4    FEC Set Index (u32 LE) — which FEC erasure coding group
```

You don't need to parse this manually — `Shred::new_from_serialized_shred()` does it for you.

#### Code Structure

**1. Bind a UDP socket**

```rust
use std::net::UdpSocket;
use solana_packet::PACKET_DATA_SIZE;

let socket = UdpSocket::bind("0.0.0.0:20000")
    .expect("Failed to bind");
```

Use `PACKET_DATA_SIZE` (1232 bytes) as your receive buffer — this is the maximum size of any Solana UDP packet.

**2. Receive packets in a loop**

```rust
let mut buf = [0u8; PACKET_DATA_SIZE];

loop {
    let (size, src) = socket.recv_from(&mut buf)?;
    // `buf[..size]` is one raw shred
}
```

Each call to `recv_from` gives you exactly one shred.

**3. Parse the shred**

```rust
use solana_ledger::shred::Shred;

let shred = Shred::new_from_serialized_shred(buf[..size].to_vec())?;
```

This validates the packet and returns a `Shred` enum (either data or code variant).

**4. Read shred fields**

```rust
shred.slot()           // u64 — which slot
shred.index()          // u32 — position in the slot
shred.shred_type()     // ShredType::Data or ShredType::Code
shred.fec_set_index()  // u32 — FEC erasure coding group
shred.data_complete()  // bool — last shred in a data segment
shred.last_in_slot()   // bool — final shred of the entire slot
shred.payload()        // &Payload (Deref<Target=[u8]>) — raw bytes
```

#### Minimal complete example

code : [https://github.com/Astralane/Shred\_example/tree/main](https://github.com/Astralane/Shred_example/tree/main)

```rust
use std::net::UdpSocket;
use solana_ledger::shred::Shred;
use solana_packet::PACKET_DATA_SIZE;

fn main() {
    let socket = UdpSocket::bind("0.0.0.0:20000").unwrap();
    let mut buf = [0u8; PACKET_DATA_SIZE];
    let mut count: u64 = 0;

    loop {
        let (size, src) = match socket.recv_from(&mut buf) {
            Ok(r) => r,
            Err(e) => { eprintln!("recv error: {e}"); continue; }
        };

        count += 1;

        match Shred::new_from_serialized_shred(buf[..size].to_vec()) {
            Ok(shred) => {
                println!(
                    "[#{count}] from={src} slot={} index={} type={:?} fec_set={}",
                    shred.slot(),
                    shred.index(),
                    shred.shred_type(),
                    shred.fec_set_index(),
                );
            }
            Err(e) => eprintln!("[#{count}] from={src} bad shred: {e}"),
        }
    }
}
```

### Latency monitor

Measure and compare end-to-end latency of your shred feed against gRPC endpoints – see who sees each transaction first.

\
`shreds-latency-probe`  listens to your UDP shred feed and connects to yellowstone-gRPC endpoints simultaneously. For every transaction, it tracks which source arrived first and measures the lag in microseconds.

\
Output: a live TUI with real-time counters and a zip archive with raw CSVs and summary tables for offline analysis.\
\
View source on [GitHub](https://github.com/Astralane/shreds-monitor).

### FAQ&#x20;

<details>

<summary>1. What is the difference between Tier 1 and Tier 2</summary>

Tiers determine how many validator-direct shred sources you receive: higher tier - more leaders covered - lower effective shred latency. Tier 2 unlocks broader leader coverage than Tier 1.

</details>

<details>

<summary>2. Which regions do you operate in?</summary>

We operate out of two regions: Frankfurt at Fra2, Terraswitch DC and New York at Ewr2.

</details>

<details>

<summary>3. Can I receive shreds on two different IPs at the same time?</summary>

Yes, this is fully supported.

</details>

<details>

<summary>4. Should I connect to shreds in both Europe and the Americas?</summary>

For maximum coverage, yes. If a leader is located in the Americas and you're only connected through our EU ingester, you'll absorb approximately 90ms of additional transit latency. That said, the majority of leaders are EU-based, so most clients get the best results staying on Frankfurt.

</details>

<details>

<summary>5. What are preconfs?</summary>

Preconfs give you a view of the chain's state before it reaches full confirmation - an earlier signal than what most providers can offer.

</details>

<details>

<summary>6. How are preconfs different from shreds?</summary>

Standard shred streams rely on the public turbine broadcast path. Our infrastructure provides visibility into validator state earlier than that path, which is what we expose as preconfs - giving you a faster read on chain state without waiting for full confirmation.

</details>

<details>

<summary>7. Why are preconfs faster than regular shred streams?</summary>

Our pipeline receives certain validators' data through a faster path than the public top-of-turbine route that most relays consume. That timing advantage is the edge we pass directly to you.

</details>

<details>

<summary>8. What is the difference between direct and turbine shreds?</summary>

Turbine is Solana's public UDP broadcast tree - shreds hop peer-to-peer, so you sit several hops from the leader. Direct shreds reach you over a shorter path with fewer hops, meaning earlier and more consistent arrival.

</details>

<details>

<summary>9. What are on-demand shreds?</summary>

Pay only for what you use - no monthly fee. Your tier is set by your trailing-24h tips. If shred quality is poor and you don't act on it, you don't tip and don't pay for that.

</details>

<details>

<summary>10. How do I pay only for the shreds I actually use?</summary>

Add the tip instruction only when an Astralane shred was the signal for a trade — so you pay strictly where it gave you an edge.\
\
**Coming soon:** the tip will attach to the closing leg of your trade, so you pay only at execution.

</details>
