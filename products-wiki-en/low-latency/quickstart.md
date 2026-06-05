---
icon: bullseye-arrow
---

# QuickStart

Low latency infrastructure delivering sub-slot transaction propagation for Solana's most demanding operators - from HFTs requiring minimal wire latency to market makers needing reliable execution during peak congestion. Along with leader path optimization, our binary-optimized mesh network provides sub-slot precision data streams.

Our dynamic priority system intelligently allocates resources for high-priority operations, backed by enterprise-grade reliability and 24/7 support.



Jump to Main sections&#x20;

{% content-ref url="endpoints-and-configs/" %}
[endpoints-and-configs](endpoints-and-configs/)
{% endcontent-ref %}

{% content-ref url="submit-transactions/" %}
[submit-transactions](submit-transactions/)
{% endcontent-ref %}



### Core Infrastructure

* Dual relay system: Transaction + Shred relays
* Direct validator integration via SwQoS-enabled connections and direct stake peering
* Advanced txn execution system for high txns with tips
* Global private fiber network with dynamic leader-aware routing
* Binary protocol optimized for minimal serialization overhead



### Performance Metrics

* 500ms latency improvement vs standard RPC nodes
* 40-50ms overhead reduction in processed transaction receipts
* Zero dropped transactions with automatic retry mechanisms
* Real-time streaming of account changes within current slot



### **Who Should Use It on Solana:**

Astralane provides specialized solutions tailored to the needs of Solana teams:

* **Fast Transaction Sending:** Essential for HFTs and Searchers who need fast transaction landing, support for multi-tx batching, rapid transaction execution.
* **More Landing Rates:** Essential for Market Makers and Solvers who want reliable transaction execution even during high network congestion times.
* **Low Latency Data Streams:** Sub-slot access to raw or parsed transaction data via redundant global paths, with support for shred-based early detection systems. Essential for bot developers seeking optimal signal detection and opportunity capture.
* **Cost Efficient Trading Setups:** Essential for trading operations seeking cost-efficient, high-throughput execution with usage-based pricing, delivering enterprise-grade performance at competitive rates.



***

#### **Custom TPU client**

Instead of going through normal RPC pipeline for transactions, we use a fast and lightweight TPU client that tracks leader schedule and sends transactions directly to the leaders also utilizing staked connection to the leaders.



**Dynamic Routing Module:**

All traffic that comes to our network are carefully analyzed and dynamically routed to our partner peers which are closest to the leader and utilize on our distributed staked network peers effectively, This is done along dedicated low latency routes which ensures the fastest propagation for the transactions.



**Smart Fee Data (Analysis)**

We help our clients optimize their transactions, through our enhanced monitoring systems which provides detailed information on how your transactions performed against different leaders, slot latency metrics.



**Retries**

By default we listen to all transactions confirmations from the network and handle transaction retries on our end to improve the landing rate, These can be modified as well to use custom retry logic as needed by clients.



**Higher Priority Txns:**

Our high-priority endpoint takes a tip to Astralane address, which in turn routes the transaction through both Jito and our staked peers and races them for the minimum possible latency, even when the current leader is running Jito validator or the native agave client



**Atomicity and Tip Refunds:**

Custom endpoints to give you a transaction hash which expires in X slots, to ensure transactions are landed within the desired slot for clients.

