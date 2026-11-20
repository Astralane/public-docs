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
1. For best performance, we recommend only sending bundles to the closest geographical region. We automatically cross-forwards bundles to all GBX builder(s) in all regions, you do not need to send bundles to all regions or to the builder(s) individually.
2. Using an IPv4 address will be **faster**
{% endhint %}

### Getting the gRPC protos: <a href="#tipping" id="tipping"></a>

The GBX protocol buffer definitions are available here:

* [https://github.com/Astralane/gbx-protos](https://github.com/Astralane/gbx-protos)

The GBX SearcherService.SendBundle gRPC and authentication flow are backwards compatible with Jito's searcher protos. This means existing Jito client code can be adapted with minimal changes: primarily updating the endpoint URL and proto imports.

### Tipping <a href="#tipping" id="tipping"></a>

For prioritization on Gravity block exchange slots, pay simply in priority fees on transactions, transfers to a designated tip accounts are not required.

Minimum priority fee of 5000 lamports. Higher fees will result in higher priority.

For oracle updates reach out to us on [discord](https://discord.gg/t8anFGy8nN) for custom submission methods with lower limits.

### Getting access <a href="#getting-access" id="getting-access"></a>

Authenticate with any keypair using the standard Jito searcher auth flow (`GenerateAuthChallenge` → `GenerateAuthTokens`), then pass the access token as a Bearer header on every request. No whitelisting needed.

Without registration you get 5 requests/s per key per region, and every bundle needs at least 5,000 lamports in priority fees. Want higher limits or no fee minimum? Ping us on [discord](https://discord.gg/t8anFGy8nN) and we'll register your pubkey.

### Finding GBX leaders

Bundles only land in slots led by validators connected to GBX. Three calls on the same endpoints tell you who those validators are and when they lead, so you can time your submissions and pick the right region. No token needed, and they don't count against your rate limit. The protos are at [github.com/Astralane/gbx-protos](https://github.com/Astralane/gbx-protos); the examples below use grpcurl from inside a clone of that repo. The responses in the examples below are all illustrative

**GetRegions**

Which region the endpoint you're talking to is in, and which regions exist.

```sh
grpcurl -plaintext -import-path protos -proto searcher.proto \
  ams-gbx.astralane.io:5064 searcher.SearcherService/GetRegions
```

Example Response:

```json
{
  "current_region": "ams",
  "available_regions": ["ams", "fra", "lon", "tyo"]
}
```

**GetConnectedLeaders**

Every validator connected to GBX and its remaining leader slots in the current epoch.

Request:

```sh
grpcurl -plaintext -import-path protos -proto searcher.proto \
  ams-gbx.astralane.io:5064 searcher.SearcherService/GetConnectedLeaders
```

Response:

```json
{
  "connected_validators": {
    "forb5u56XgvzxiKfRt4FVNFQKJrd2LWAfNCsCqL6P7q": {
      "slots": [471782752, 471782753, 471782754, 471782755, ...]
    },
    "C8Bey3LKVJHVqN6xPTeW8WJfUgFQAeGNBpT4Rp99JP1k": {
      "slots": [471756396, 471756397, 471756398, 471756399, ...]
    },
    "Ha1iade1AH3B12K9SccfWoPdFtQKKQsj2ZyWwxcjqJJU": {
      "slots": [471756452, 471756453, 471756454, 471756455, ...]
    }
  }
}
```

**GetConnectedLeadersRegioned**

The same data grouped by the region that serves each validator. Send a bundle for a given slot to that leader's region for the lowest latency.

Request: a list of regions, or empty for all.



```sh
grpcurl -plaintext -import-path protos -proto searcher.proto \
  -d '{"regions": ["ams", "tyo"]}' \
  ams-gbx.astralane.io:5064 searcher.SearcherService/GetConnectedLeadersRegioned
```

Response:

```json
{
  "connected_validators": {
    "ams": {
      "connected_validators": {
        "forb5u56XgvzxiKfRt4FVNFQKJrd2LWAfNCsCqL6P7q": {
          "slots": [471782752, 471782753, 471782754, 471782755, ...]
        }
      }
    },
    "tyo": {
      "connected_validators": {
        "Ha1iade1AH3B12K9SccfWoPdFtQKKQsj2ZyWwxcjqJJU": {
          "slots": [471756452, 471756453, 471756454, 471756455, ...]
        }
      }
    }
  }
}
```

### Current Features <a href="#getting-access" id="getting-access"></a>

#### 1. Revert Protection

The GBX interface provides revert protection for submitted bundles. If any transaction reverts, the entire bundle is dropped and does not land on-chain.

Bundles using `dropping_indices` are an exception. A listed transaction may be omitted if it fails, while the remaining transactions proceed.

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

Upcoming GBX slots come from `GetConnectedLeaders` (see Finding GBX leaders above).

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

#### More features? <a href="#getting-access" id="getting-access"></a>

For custom support for oracle updates, ACE, etc. reach out to us on our [discord server](https://discord.gg/t8anFGy8nN).
