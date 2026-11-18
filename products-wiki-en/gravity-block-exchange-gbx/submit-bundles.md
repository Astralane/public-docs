---
description: >-
  Astralane hosts and manages a broadcast endpoint which submits bundles to all
  competing builder(s) on the Gravity Block Exchange marketplace.
icon: paper-plane
---

# Submit Bundles

### Participating Builders

All specifications provided on this page work across the following participating builders in GBX:

1. Bifrost - [https://bifrostbuilder.com/](https://bifrostbuilder.com/)
2. Titan Builder - [https://www.titanbuilder.xyz/](https://www.titanbuilder.xyz/)

***

### Endpoints

<table><thead><tr><th width="139.34375">Region</th><th>gRPC Endpoint</th></tr></thead><tbody><tr><td>Frankfurt</td><td><a href="http://fra-gbx.astralane.io:5064/">http://fra-gbx.astralane.io:5064</a></td></tr><tr><td>Amsterdam</td><td><a href="http://ams-gbx.astralane.io:5064/">http://ams-gbx.astralane.io:5064</a></td></tr><tr><td>London</td><td><a href="http://lon-gbx.gateway.astralane.io:5064/">http://lon-gbx.astralane.io:5064</a></td></tr><tr><td>Tokyo</td><td><a href="http://jp-gbx.astralane.io:5064/">http://jp-gbx.astralane.io:5064</a></td></tr></tbody></table>

{% hint style="info" %}
1. For best performance, we recommend only sending bundles to the closest geographical region. We automatically cross-forwards bundles to all GBX builder(s) in all regions, you do not need to send bundles to all regions or to the builder(s) individually.&#x20;
2. Using an IPv4 address will be **faster**
{% endhint %}

### Getting the gRPC protos: <a href="#tipping" id="tipping"></a>

The GBX protocol buffer definitions are available here:

* &#x20;[https://github.com/Astralane/gbx-protos](https://github.com/Astralane/gbx-protos)&#x20;

The GBX SearcherService.SendBundle gRPC and authentication flow are backwards compatible with Jito's searcher protos. This means existing Jito client code can be adapted with minimal changes: primarily updating the endpoint URL and proto imports.&#x20;

### Tipping  <a href="#tipping" id="tipping"></a>

For prioritization on Gravity block exchange slots, pay simply in priority fees on transactions, transfers to a designated tip accounts are not required.

Minimum priority fee of 5000 lamports. Higher fees will result in higher priority.&#x20;

For oracle updates out to us on [discord](https://discord.gg/t8anFGy8nN) for custom submission methods with lower limits.&#x20;

### Getting access <a href="#getting-access" id="getting-access"></a>

Before you can send bundles through the authenticated `searcher.SearcherService`, your searcher pubkey must be whitelisted by the Astralane team.

Authenticated searchers get higher rate limits. If the public rate limits are enough for your flow, public submission over `searcher.BundleService` or JSON-RPC is also available without authentication.

### Current Features <a href="#getting-access" id="getting-access"></a>

#### 1. Revert Protection&#x20;

The above interface for GBX provides revert protection for submitted bundles. If any transaction in your bundle reverts, the entire bundle is dropped and will not land on-chain.&#x20;

#### 2. Target Slot for execution

You can specify the slot number which you want your txn to execute in when sending bundles to our interface. It will only execute in that particular slot.

```
message Bundle {
  shared.Header header = 2;
  repeated packet.Packet packets = 3;
  ....
  optional uint64 slot = 5;
  .....
}
```

#### 3. Dropping Indices

Transactions in bundles can be marked as droppable for partial atomicity. If one fails, the GBX builder(s) drops just that member instead of voiding the whole bundle.

`droppingIndices` lists the positions in the bundle the GBX builder(s) is **permitted** to omit. A member listed there is dropped only if it actually fails - either because it errored in the VM, or because its signature was already applied earlier in the window.

```
message Bundle {
  shared.Header header = 2;
  repeated packet.Packet packets = 3;
  ....
  repeated uint32 dropping_indices = 4;
  .....
}
```

Some example scenarios:

```json
["txA", "txB", "txC", "txD"]      // params[0], executes in this order
"droppingIndices": []             // all four should succeed — classic atomic bundle
"droppingIndices": [1]            // only txB may be omitted
"droppingIndices": [0, 2]         // txA and txC omittable; txB, txD should succeed
"droppingIndices": [3]            // only the last may be omitted
"droppingIndices": [0,1,2,3]      // all omittable — required for mevVariant 4
```

#### 4. Target Accounts

Target accounts can be specified when submitting bundles, this lets the GBX builder(s) know which accounts the bundle intends to affect, helping it identify and simulate them when state in these accounts changes.

For example: You want to do a backrun after a change in AMM pool account, you can specify the pool account you are targeting, the bundle will be simulated after a write on that account. Limit the number of accounts specified to the absolute necessary in order to ensure better scheduling outcomes

```
message Bundle {
  shared.Header header = 2;
  repeated packet.Packet packets = 3;
  ...
  repeated bytes target_accounts = 6;
  ...
}
```

**Important Points:**

* max target\_account: 16
* for highly contended accounts, this works on a best effort basis by the GBX builder(s)

#### **5. `jitodontfront` support**

All GBX builders will support the `jitodontfront` memo convention, allowing transactions that include this flag to receive front-running protection. Bundles attempting to front-run flagged transactions will be rejected by the builders, giving applications and traders an additional layer of protection. We follow standards set by builders when interacting with the Solana ecosystem if any additional ix marker restrictions are required, kindly do reach out to us.

#### Rate Limits <a href="#getting-access" id="getting-access"></a>

There are no public rate limits on the above GBX endpoints. Kindly reach out to the team via our [discord server](https://discord.gg/t8anFGy8nN)&#x20;

#### More features? <a href="#getting-access" id="getting-access"></a>

For custom support for oracle updates, ACE, etc. reach out to us on our [discord server](https://discord.gg/t8anFGy8nN).
