---
description: Access on chain data streams at lightning high speeds
hidden: true
noIndex: true
icon: binoculars
---

# Data Streams

### **Dedicated gRPC streams**

gRPC streams provide highly configurable, real-time data streams and are the fastest way to stream Solana data to your backend.



**Our Dedicated Endpoints:**

Tokyo: 45.250.255.183,189.1.164.31(backup rpc)

FRA: 162.19.222.232,79.137.101.98 (backup rpc)



**Accessing the Co-located Server**

```bash
IP: (Will be provided )
Username: ubuntu
ssh key will be added once you give it to us
ssh -i <ssh key> ubuntu@IP
```



**Co-located Server Specs**

Intel Xeon-E 2136 - 6c/12t - 3.3 GHz/4.5 GHz

32 GB ram

500Mbps upload and download speed



**Accessing the RPC**

Once you get access to the test server you can access to the RPC via a private network

```bash
IP address : 192.168.1.77
HTTP RPC port: 8899
HTTP GRPC port: 10000
Websocket Port: 8900
Websocket Geyser Plugin: 10050
```



**Please note:**

1. The RPC has SWQoS enabled with our [validator](https://www.jito.network/validator/sShosKd6uA5c1ZpVMxdsE6do13TLRWSMYsXbSMmNC77/) and as of now the requests made to the RPC are not rate limited
2. The RPC and the co-located server provided to you is located in Frankfurt, Germany and the ping the Jito Block Engine is:

<figure><img src="../.gitbook/assets/Screenshot 2024-12-04 at 7.25.55 PM.png" alt="" width="563"><figcaption></figcaption></figure>



**Available Subscriptions:**

* Accounts Streams
* Transactions Streams
* Block Streams
* Slot Streams

_If block/account subscribe or other streams are required do please let us know_



**Sanity checks**

* Ping 192.168.1.77 - for internal network access
* Ping 8.8.8.8
*   Check if rpc port is working

    ```bash
    solana -u [<http://192.168.1.77:8899>](<http://rpc:8899/>) epoch-info 
    ```
*   Using solana catchup CLI to check if the RPC is synced (only needed if we perform upgrades):

    ```bash
    solana catchup -um CbvjseFvBqvBFz3Xe75iuA9vhkkFrjmnYSPx5AEcojwp <http://192.168.1.77:8899>
    ```



**Client Code references for grpc (in rust)**

1. For a reference implementation of JITO gRPC client, see: [https://github.com/jito-foundation/geyser-grpc-plugin/blob/master/cli/src/main.rs](https://github.com/jito-foundation/geyser-grpc-plugin/blob/master/cli/src/main.rs)
2. For a reference implementation of yellowstone gRPC client, see: [https://github.com/rpcpool/yellowstone-grpc/tree/master/examples/rust](https://github.com/rpcpool/yellowstone-grpc/tree/master/examples/rust)
3. For a reference implementation of Astralane custom low-latency gRPC client, see: [Astralane gRPC Client Examples](https://github.com/Astralane/astralane-streaming-client/tree/fe46f26b80f23295646f493e9bfbed0fb267fa77/examples/). These example demonstrates best practices for maintaining persistent connections, efficient message handling, and minimizing processing overhead.
4. Triton docs on subscribing to requests - [https://docs.triton.one/project-yellowstone/dragons-mouth-grpc-subscriptions#grpcurl](https://docs.triton.one/project-yellowstone/dragons-mouth-grpc-subscriptions#grpcurl)
5. For benchmarking and latency testing, see; [https://github.com/Astralane/Astralane-basic-benchmark/blob/main/solana\_txn/src/main.rs](https://github.com/Astralane/Astralane-basic-benchmark/blob/main/solana_txn/src/main.rs)

_If your team needs more client examples for different languages or help with some of the subcription requests or adding cusotmization’s or with protobuf’s, dont hesitate to reachout._



### 2.2. Enriched Web sockets +Geyser

Websocket stream utilizing geysers for more enriched data than on native solana rpc. with support for `transactionSubscribe` and `accountSubscribe`.



**Transaction Subscribe:**

The transactionSubscribe websocket method enables real-time transaction events.

`vote`: A boolean flag to include/exclude vote-related transactions.

`failed`: A boolean flag to include/exclude transactions that failed.

`signature`: Filters updates to a specific transaction based on its signature.

`accountInclude`: A list of accounts for which you want to receive transaction updates. This means that only one of the accounts must be included in the transaction updates (e.g., Account 1 OR Account 2).

`accountExclude`: A list of accounts you want to exclude from transaction updates.

`accountRequired`: Transactions must involve these specified accounts to be included in updates.



**Accounts Subscribe:**

This method aligns directly with the Solana [Websocket API specification](https://solana.com/docs/rpc/websocket#accountsubscribe).



**Confirmed Signature subscribe**

It is not scalable to call signature subscribe for if you are a large no: of sending transactions , The confirmed transactions is an enhancement on top the transactions subscribe which can give you only the signatures of the transactions belonging to an account

`accountInclude`: A list of accounts for which you want to receive transaction updates. This means that only one of the accounts must be included in the transaction updates (e.g., Account 1 OR Account 2).
