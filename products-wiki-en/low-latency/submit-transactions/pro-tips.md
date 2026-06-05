---
icon: bolt
---

# Pro Tips

## Keep-Alive Connections

Keep-Alive, also known as persistent connections, is a feature used in networking protocols like HTTP and TCP to maintain an open connection between a client and server for multiple requests and responses, rather than creating a new connection for each interaction.

### Purpose

The main goal of Keep-Alive is to reduce latency and overhead by avoiding the need to repeatedly open and close connections. This is particularly useful in high-performance systems or when handling multiple short-lived requests.

We provide different type of Get health method to ensure smooth integration with your http clients

### Get Health

{% tabs %}
{% tab title="JSON RPC" %}
```bash
curl --location 'http://fr.gateway.astralane.io/iris' \
--header 'api_key: APIKEY' \
--header 'Content-Type: application/json' \
--data '{
     "jsonrpc": "2.0",
     "id": 1,
     "method": "getHealth"
   }
'
```
{% endtab %}

{% tab title="GET method" %}
```bash
curl --location 'http://fr.gateway.astralane.io/gethealth'
```
{% endtab %}

{% tab title="/irisb" %}
```bash
curl --location --request POST 'http://fr.gateway.astralane.io/irisb?api-key=APIKEY&method=getHealth'
```
{% endtab %}

{% tab title="/iris2" %}
```bash
curl --location --request POST 'http://fr.gateway.astralane.io/iris2?api-key=APIKEY&method=getHealth'
```
{% endtab %}
{% endtabs %}

### Key Features

* Connection Reuse: A single TCP connection is reused for multiple requests, eliminating the setup and teardown cost of new connections.
* Reduced Latency: Avoids the round-trip time needed to establish new TCP handshakes, improving overall response time.
* Improved Throughput: Keeps the pipeline open, allowing for more requests to be served efficiently over the same connection.
* Lower Resource Consumption: Reduces CPU and memory usage on both client and server by minimizing connection churn.
* Heartbeat or Ping Mechanism: Many implementations use periodic signals to keep the connection from being considered idle or closed by intermediate network devices.

```rust
async fn send_txn_with_keep_alive() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::builder()
        .pool_idle_timeout(Some(Duration::from_secs(85))) // Keep connections alive for 85
        .build()?;
```

Here is the [full source with example](https://github.com/Astralane/keep-Alive-Example/blob/main/src/main.rs)<br>

## Wallet Rotation

Tip wallets are prone to write locks, which may result in landing of your transactions being delayed.&#x20;

We recommend rotating our tip wallets to avoid this issue.&#x20;
