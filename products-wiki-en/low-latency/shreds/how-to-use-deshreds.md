# How to use Deshreds

1\. Connection

| Parameter | Value                                                                                      |
| --------- | ------------------------------------------------------------------------------------------ |
| Protocol  | gRPC (HTTP/2), plaintext or TLS — depends on the entry point                               |
| Service   | `geyser.Geyser`                                                                            |
| Method    | `SubscribeDeshred(stream SubscribeDeshredRequest) returns (stream SubscribeUpdateDeshred)` |
| Address   | issued with your API key (`deshred.bind_addr` in the hub config, e.g. `1.2.3.4:52000`)     |

**Authorization**

Pass your API key in the request metadata, either way:

```
x-token: <API_KEY>
```

or

```
authorization: Bearer <API_KEY>
```

This is the same key used for other Astralane services. Missing or unknown keys are rejected with `UNAUTHENTICATED`.



Exceeding it returns `RESOURCE_EXHAUSTED` (`deshred subscriber limit reached [for api key]`). The slot is freed as soon as the stream closes.

***

#### 2. Protobuf

```proto
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import public "solana-storage.proto";

package geyser;

service Geyser {
  rpc SubscribeDeshred(stream SubscribeDeshredRequest) returns (stream SubscribeUpdateDeshred) {}
}

message SubscribeDeshredRequest {
  map<string, SubscribeRequestFilterDeshredTransactions> deshred_transactions = 1;
  optional SubscribeRequestPing ping = 2;
}

message SubscribeRequestFilterDeshredTransactions {
  optional bool vote = 1;
  repeated string account_include = 2;
  repeated string account_exclude = 3;
  repeated string account_required = 4;
}

message SubscribeRequestPing { int32 id = 1; }

message SubscribeUpdateDeshred {
  repeated string filters = 1;
  oneof update_oneof {
    SubscribeUpdateDeshredTransaction deshred_transaction = 2;
    SubscribeUpdatePing ping = 3;
    SubscribeUpdatePong pong = 4;
    SubscribeUpdateSlot slot = 6;
  }
  google.protobuf.Timestamp created_at = 5;
}

message SubscribeUpdateDeshredTransaction {
  SubscribeUpdateDeshredTransactionInfo transaction = 1;
  uint64 slot = 2;
}

message SubscribeUpdateDeshredTransactionInfo {
  bytes signature = 1;
  bool is_vote = 2;
  solana.storage.ConfirmedBlock.Transaction transaction = 3;
  repeated bytes loaded_writable_addresses = 4;
  repeated bytes loaded_readonly_addresses = 5;
  uint32 completed_data_set_starting_shred_index = 6;
  uint32 completed_data_set_ending_shred_index_exclusive = 7;
}
```

In practice the server only ever sends two variants of `update_oneof`: `deshred_transaction` and `pong`. `ping`/`slot` are reserved for future use.&#x20;

***

#### 3. Filters

Filters are named, same as Yellowstone. The first message on the stream sets your subscription - **without at least one filter, the stream stays silent.**

An empty filter means "everything":

```json
{ "deshred_transactions": { "all": {} } }
```

Fields inside one filter combine with **AND**:

| Field              | Meaning                                                                             |
| ------------------ | ----------------------------------------------------------------------------------- |
| `vote`             | `true` - vote transactions only, `false` - non-vote only, unset — either            |
| `account_include`  | transaction references **at least one** of the listed accounts (empty list = no-op) |
| `account_exclude`  | transaction references **none** of the listed accounts                              |
| `account_required` | transaction references **all** of the listed accounts                               |

Both static message keys and addresses loaded from ALTs (`loaded_writable_addresses` / `loaded_readonly_addresses`) are checked.

Multiple filters run independently; each update's `filters` field lists the names of every filter that matched (alphabetically sorted). If nothing matches, no update is sent.

Examples:

```jsonc
// entire stream, no votes
{ "deshred_transactions": { "nonvote": { "vote": false } } }

// anything touching a wallet
{ "deshred_transactions": { "wallet": { "account_include": ["<PUBKEY>"] } } }

// direct Raydium swaps, excluding Jupiter routing
{ "deshred_transactions": { "direct": {
    "account_include": ["<RAYDIUM>"],
    "account_exclude": ["<JUPITER>"] } } }

// trader and venue in the same transaction
{ "deshred_transactions": { "pair": {
    "account_required": ["<TRADER>", "<VENUE>"] } } }
```

**Updating the subscription on the fly**

The stream is bidirectional - send a new `SubscribeDeshredRequest` on an already-open connection at any time:

* a request with a non-empty `deshred_transactions` **fully replaces** the current filter set (it doesn't merge)
* a request with an empty `deshred_transactions` and **no** `ping` clears all filters, and the stream goes quiet
* a request with **only** `ping` leaves filters untouched; you get back a `pong` with the same `id` and the current filter names in `filters`
* a request with both filters **and** `ping` replaces the filters and replies with `pong`

An invalid base58 pubkey in any filter returns `INVALID_ARGUMENT` naming the filter and field - **the stream is closed**, and the old filters stop applying. Validate keys client-side before sending.

***

#### 4. What's in an update

* `slot` - the slot whose shreds this transaction was reconstructed from
* `signature` - the first signature, as raw bytes (not base58)
* `is_vote` - whether this is a vote transaction
* `transaction` - a standard `solana.storage.ConfirmedBlock.Transaction`: signatures, header, `account_keys`, `recent_blockhash`, `instructions`, `versioned`, `address_table_lookups`
* `loaded_writable_addresses` / `loaded_readonly_addresses` - resolved ALT addresses. Only populated if ALT caching is enabled on the service **and** the table is already cached; otherwise these are empty and the transaction is still delivered
* `completed_data_set_*` - the shred indices of the FEC set this transaction was assembled from (useful for debugging/dedup)
* `created_at` - server-side timestamp when the update was produced

**Semantics to keep in mind**

1. This is **pre-execution** data - no confirmation or finality guarantee. Verify landing via RPC.
2. No `meta`, execution status, or balances - only the transaction itself.
3. Ordering across slots isn't strictly monotonic; key off the `slot` field.
4. Duplicates are possible - dedupe by `signature` in a sliding window.
5. Transactions arrive in batches as FEC sets complete, not one at a time.

***

#### 5. Errors & reconnection

| Code                                                                    | When                                           | What to do                               |
| ----------------------------------------------------------------------- | ---------------------------------------------- | ---------------------------------------- |
| `UNAUTHENTICATED`                                                       | missing/unknown API key                        | check metadata, don't retry in a loop    |
| `RESOURCE_EXHAUSTED` `...limit reached`                                 | stream limit exceeded                          | close excess streams, retry with backoff |
| `RESOURCE_EXHAUSTED` `...lagged by N updates` / `...outbound lag limit` | client isn't keeping up                        | reconnect, speed up processing           |
| `UNAVAILABLE` `warming the ALT cache`                                   | service is warming its ALT cache after startup | retry after a few seconds                |
| `INVALID_ARGUMENT`                                                      | malformed pubkey in a filter                   | fix the filter                           |

**Lag is the most common thing that breaks clients.** Each subscriber has a bounded queue. If your handler is slow (parsing, DB writes, blocking I/O in the read loop), the server drops the stream with `RESOURCE_EXHAUSTED`. Keep the read loop to receiving and forwarding into your own queue - do the actual work in separate workers.

Recommended policy: exponential backoff from 100ms to 5s, fully recreate the stream and resend filters on reconnect, keepalive-ping every 10-30s (also enable gRPC HTTP/2 keepalive).

***

#### 6. Example: Rust (tonic)

```toml
tonic = "0.14"
prost = "0.14"
tokio = { version = "1", features = ["full"] }
tokio-stream = "0.1"
```

```rust
use std::collections::HashMap;

use shreds_proto::geyser::{
    geyser_client::GeyserClient, SubscribeDeshredRequest,
    SubscribeRequestFilterDeshredTransactions, subscribe_update_deshred::UpdateOneof,
};
use tonic::{metadata::MetadataValue, transport::Channel, Request};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let api_key = std::env::var("ASTRALANE_API_KEY")?;
    let channel = Channel::from_static("http://HOST:52000").connect().await?;

    let token: MetadataValue<_> = api_key.parse()?;
    let mut client = GeyserClient::with_interceptor(channel, move |mut req: Request<()>| {
        req.metadata_mut().insert("x-token", token.clone());
        Ok(req)
    });

    let subscribe = SubscribeDeshredRequest {
        deshred_transactions: HashMap::from([(
            "wallet".to_string(),
            SubscribeRequestFilterDeshredTransactions {
                vote: Some(false),
                account_include: vec!["<PUBKEY>".to_string()],
                ..Default::default()
            },
        )]),
        ping: None,
    };

    let (tx, rx) = tokio::sync::mpsc::channel(8);
    tx.send(subscribe).await?;

    let mut stream = client
        .subscribe_deshred(tokio_stream::wrappers::ReceiverStream::new(rx))
        .await?
        .into_inner();

    while let Some(update) = stream.message().await? {
        match update.update_oneof {
            Some(UpdateOneof::DeshredTransaction(tx_update)) => {
                let info = tx_update.transaction.unwrap();
                println!(
                    "slot={} sig={} filters={:?}",
                    tx_update.slot,
                    bs58::encode(&info.signature).into_string(),
                    update.filters,
                );
            }
            Some(UpdateOneof::Pong(pong)) => println!("pong {}", pong.id),
            _ => {}
        }
    }
    Ok(())
}
```

***

#### 7. Example: Python (grpcio)

```sh
pip install grpcio grpcio-tools base58
python -m grpc_tools.protoc -I proto \
  --python_out=. --grpc_python_out=. \
  proto/geyser.proto proto/solana-storage.proto
```

```python
import base58, grpc, queue
import geyser_pb2 as pb, geyser_pb2_grpc as pb_grpc

API_KEY = "..."
ENDPOINT = "HOST:52000"

def requests(q):
    req = pb.SubscribeDeshredRequest()
    req.deshred_transactions["wallet"].account_include.append("<PUBKEY>")
    req.deshred_transactions["wallet"].vote = False
    yield req
    while True:
        yield q.get()

def main():
    channel = grpc.insecure_channel(ENDPOINT, options=[
        ("grpc.keepalive_time_ms", 20000),
        ("grpc.max_receive_message_length", 64 * 1024 * 1024),
    ])
    stub = pb_grpc.GeyserStub(channel)
    q = queue.Queue()

    for update in stub.SubscribeDeshred(requests(q), metadata=(("x-token", API_KEY),)):
        which = update.WhichOneof("update_oneof")
        if which == "deshred_transaction":
            t = update.deshred_transaction
            sig = base58.b58encode(t.transaction.signature).decode()
            print(t.slot, sig, list(update.filters))
        elif which == "pong":
            print("pong", update.pong.id)

if __name__ == "__main__":
    main()
```
