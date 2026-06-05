---
icon: jet-fighter
---

# QUIC Transaction Submission

Astralane's QUIC endpoint provides a faster, more efficient path for submitting Solana transactions compared to traditional HTTP/JSON-RPC.

### Available Endpoints

* <table><thead><tr><th width="255.17578125">Location</th><th>IP Address</th><th>MEV Protected IP address</th></tr></thead><tbody><tr><td>Frankfurt (Recommended)</td><td><code>185.191.117.97:7000</code></td><td><code>185.191.117.97:9000</code></td></tr><tr><td>Frankfurt</td><td><code>45.139.132.160:7000</code></td><td><code>45.139.132.160:9000</code></td></tr><tr><td>San Francisco</td><td><code>74.118.142.151:7000</code></td><td><code>74.118.142.151:9000</code></td></tr><tr><td>Tokyo</td><td><code>189.1.164.31:7000</code></td><td><code>189.1.164.31:9000</code></td></tr><tr><td>New York</td><td><code>64.130.45.19:7000</code></td><td><code>64.130.45.19:9000</code></td></tr><tr><td>Amsterdam (Recommended)</td><td><code>64.130.43.43:7000</code></td><td><code>64.130.43.43:9000</code></td></tr><tr><td>Amsterdam</td><td><code>84.32.186.73:7000</code></td><td><code>84.32.186.73:9000</code></td></tr><tr><td>Limburg</td><td><code>162.19.222.232:7000</code></td><td><code>162.19.222.232:9000</code></td></tr><tr><td>Singapore</td><td><code>67.209.54.176:7000</code></td><td><code>67.209.54.176:9000</code></td></tr><tr><td>Lithuania</td><td><code>84.32.97.47:7000</code></td><td><code>84.32.97.47:9000</code></td></tr></tbody></table>

### JSON-RPC VS QUIC

| Parameter             | HTTP/JSON-RPC                                              | QUIC                                                     |
| --------------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| **Latency**           | HTTP overhead + JSON serialization/deserialization         | Direct binary stream, sub-millisecond send               |
| **Connection**        | New TCP connection per request (or keep-alive with limits) | Single persistent QUIC connection, multiplexed streams   |
| **Concurrency**       | Sequential requests or multiple connections                | Up to 64 concurrent streams on a single connection       |
| **Serialization**     | JSON-RPC wrapping + base64/base58 encoding                 | Raw bincode bytes; no encoding overhead                  |
| **Reconnection**      | Manual retry logic required                                | Automatic transparent reconnection built into the client |
| **Protocol overhead** | TCP + TLS handshake per connection                         | 0-RTT capable, built-in encryption                       |

### Why Use `astralane-quic-client`?

* **Zero-config authentication** : TLS certificate with your API key is generated automatically
* **Automatic reconnection** : transparent recovery from timeouts and server restarts
* **Fire-and-forget** : no response parsing or callback handling needed
* **Connection keep-alive** : persistent connection with automatic 25s keep-alive pings, no repeated handshakes
* **Built-in backpressure :** stream limits are handled gracefully without dropping your connection

Only **authentication errors** (`UNKNOWN_API_KEY`) require manual intervention, verify your API key is correct and active.

### Getting Started

The recommended way to interact with Astralane's QUIC endpoint is through the official [astralane-quic-client](https://github.com/Astralane/astralane-quic-client).

#### Installation

```toml
[dependencies]
astralane-quic-client = { git = "https://github.com/Astralane/astralane-quic-client" }
```

#### Quick Example

```rust
use astralane_quic_client::AstralaneQuicClient;

let client = AstralaneQuicClient::connect("lim.gateway.astralane.io:7000", "your-api-key").await?;

// Build and serialize your Solana transaction
let tx_bytes = bincode::serialize(&transaction)?;

// Send (fire-and-forget, auto-reconnects on failure)
client.send_transaction(&tx_bytes).await?;

// Small delay before close to let the server read in-flight streams
tokio::time::sleep(Duration::from_millis(100)).await;
client.close().await;
```

***

### Server Limits

| Parameter                             | Value      | Description                                                           |
| ------------------------------------- | ---------- | --------------------------------------------------------------------- |
| Max connections per API key           | 10         | Create multiple client instances for more connections                 |
| Max concurrent streams per connection | 64         | Streams beyond this limit will block until a slot frees up            |
| Stream read timeout                   | 750 ms     | Server closes the stream if transaction bytes aren't received in time |
| Max transaction size                  | 1232 bytes | Standard Solana transaction size limit                                |
| Idle timeout                          | 30 s       | Connection closes if no activity; client sends keep-alive every 25s   |

***

### Error Codes

The server may close your connection with these application-level error codes:

| Code | Name               | When It Happens                                                                                        |
| ---- | ------------------ | ------------------------------------------------------------------------------------------------------ |
| 0    | `OK`               | Normal closure (client or server initiated)                                                            |
| 1    | `UNKNOWN_API_KEY`  | API key not recognized (check your API key is valid and active)                                        |
| 2    | `CONNECTION_LIMIT` | Too many connections for this API key (close unused connections or contact support for a higher limit) |

#### Rate Limiting

When the rate limit is exceeded, the server **silently drops** excess transactions. The connection stays alive - no error is returned to the client. Monitor your transaction landing rate to detect if you're being rate limited.

#### Stream Limits

When the concurrent stream limit (64) is reached, the client will block (backpressure) until a stream slot frees up. The server does not close the connection, your send call will simply take longer to return.
