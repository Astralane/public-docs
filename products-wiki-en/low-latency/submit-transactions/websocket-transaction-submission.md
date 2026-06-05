---
icon: jet-fighter
---

# WebSocket Transaction Submission

Astralane's WebSocket endpoint provides a fast, browser-native path for submitting Solana transactions. Designed for frontends and trading terminals where QUIC (mTLS) isn't available.

### Available Endpoints

<table><thead><tr><th width="296.921875">Location</th><th>Endpoint</th></tr></thead><tbody><tr><td>Frankfurt (Recommended)</td><td><code>wss://fr.gateway.astralane.io:5500</code></td></tr><tr><td>Frankfurt</td><td><code>wss://fr2.gateway.astralane.io:5500</code></td></tr><tr><td>San Francisco</td><td><code>wss://la.gateway.astralane.io:5500</code></td></tr><tr><td>Tokyo</td><td><code>wss://jp.gateway.astralane.io:5500</code></td></tr><tr><td>New York</td><td><code>wss://ny.gateway.astralane.io:5500</code></td></tr><tr><td>Amsterdam (Recommended)</td><td><code>wss://ams.gateway.astralane.io:5500</code></td></tr><tr><td>Amsterdam 2</td><td><code>wss://ams2.gateway.astralane.io:5500</code></td></tr><tr><td>Limburg</td><td><code>wss://lim.gateway.astralane.io:5500</code></td></tr><tr><td>Singapore</td><td><code>wss://sg.gateway.astralane.io:5500</code></td></tr><tr><td>Lithuania</td><td><code>wss://lit.gateway.astralane.io:5500</code></td></tr></tbody></table>

### Available Paths

<table><thead><tr><th width="280.58203125">Path</th><th>Description</th></tr></thead><tbody><tr><td><code>/iris</code></td><td>Standard transaction submission</td></tr><tr><td><code>/mev-protect</code></td><td>MEV-protected transaction submission</td></tr></tbody></table>

Example URLs:

```
wss://fr.gateway.astralane.io:5500/iris?api_key=your-api-key
wss://fr.gateway.astralane.io:5500/mev-protect?api_key=your-api-key
```

### HTTP/JSON-RPC vs WebSocket

<table><thead><tr><th width="200.87109375">Parameter</th><th>HTTP/JSON-RPC</th><th>WebSocket</th></tr></thead><tbody><tr><td><strong>Latency</strong></td><td>HTTP overhead + JSON serialization per request</td><td>Single handshake, then raw binary frames</td></tr><tr><td><strong>Connection</strong></td><td>New round trip per transaction</td><td>Persistent - one connection, unlimited transactions</td></tr><tr><td><strong>Serialization</strong></td><td>JSON-RPC wrapping + base64 encoding</td><td>Raw binary transaction bytes, no encoding</td></tr><tr><td><strong>CORS</strong></td><td>Requires CORS headers, preflight requests</td><td>No CORS restrictions - works from any domain</td></tr><tr><td><strong>Response model</strong></td><td>Synchronous - client waits for JSON-RPC response</td><td>Fire-and-forget - no response, no blocking</td></tr><tr><td><strong>Browser support</strong></td><td><code>fetch()</code> API</td><td>Native <code>new WebSocket()</code></td></tr><tr><td><strong>Dead connection</strong></td><td>Not detected - ephemeral connections</td><td>Auto-detected via ping/pong heartbeat</td></tr><tr><td><strong>Protocol overhead</strong></td><td>HTTP headers (~500 bytes) per request</td><td>~6 bytes WebSocket framing per message</td></tr></tbody></table>

### Why Use WebSocket?

* **Browser-native**: Works directly from any browser with `new WebSocket()` - no libraries, no backend proxy needed
* **Zero CORS issues**: WebSocket connections bypass CORS entirely, unlike HTTP endpoints
* **Persistent connection**: One handshake, then send unlimited transactions with minimal framing overhead
* **Raw binary**: Send serialized Solana transaction bytes directly - no base64 encoding, no JSON-RPC wrapping
* **Fire-and-forget**: Send and move on, no response parsing needed
* **Dead connection detection**: Server sends ping every 25s, automatically cleans up dead connections

### Important: Fire-and-Forget

Unlike JSON-RPC endpoints (e.g., `/iris`) which return a transaction signature in the response, the WebSocket endpoint does **not** return any response. Transactions are forwarded silently with no signature, no acknowledgment, no error message is sent back to the client.

This means:

* **You must track signatures client-side:** since you sign the transaction before sending, you already have the signature.
* **Rate-limited transactions are silently dropped:** the connection stays alive, but excess transactions are not forwarded. Monitor your transaction landing rate to detect if you're being rate limited.
* **Invalid transactions are silently dropped:** transactions exceeding the size limit or failing validation are discarded without notification.

### Getting Started

{% tabs %}
{% tab title="Rust" %}
```rust
use tokio_tungstenite::connect_async;
use tungstenite::Message;
use futures_util::{SinkExt, StreamExt};

let (ws_stream, _) = connect_async("wss://fr.gateway.astralane.io:5500/iris?api_key=your-api-key")
    .await
    .expect("Failed to connect");

let (mut sender, mut receiver) = ws_stream.split();

// Spawn a read task — this is required so that tungstenite flushes
// automatic pong responses to the server's pings.
tokio::spawn(async move {
    while let Some(_) = receiver.next().await {}
});

// Build and serialize your Solana transaction
let tx_bytes = bincode::serialize(&transaction)?;

// Send raw binary (fire-and-forget)
sender.send(Message::Binary(tx_bytes.into())).await?;
```
{% endtab %}

{% tab title="TypeScript / React" %}
```typescript
import { Transaction, Connection, Keypair } from '@solana/web3.js';

function createAstralaneWs(endpoint: string, apiKey: string, path: string = '/iris'): WebSocket {
  const ws = new WebSocket(`wss://${endpoint}:5500${path}?api_key=${apiKey}`);
  ws.binaryType = 'arraybuffer';
  return ws;
}

// Connect
const ws = createAstralaneWs('fr.gateway.astralane.io', 'your-api-key');

ws.onopen = () => {
  // Send transactions as raw binary
  const tx = buildTransaction(); // your transaction builder
  ws.send(tx.serialize());
};
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const ws = new WebSocket('wss://fr.gateway.astralane.io:5500/iris?api_key=your-api-key');
ws.binaryType = 'arraybuffer';

ws.onopen = () => {
  // Build and serialize your Solana transaction
  const transaction = new Transaction();
  // ... add instructions, sign ...
  const serializedTx = transaction.serialize();

  // Send raw binary (fire-and-forget)
  ws.send(serializedTx);
};

ws.onclose = (event) => {
  console.log(`Connection closed: code=${event.code}`);
};
```
{% endtab %}

{% tab title="Python" %}
```python
import asyncio
import ssl
import websockets

async def send_transaction(tx_bytes: bytes):
    uri = "wss://fr.gateway.astralane.io:5500/iris?api_key=your-api-key"

    async with websockets.connect(uri) as ws:
        await ws.send(tx_bytes)
```
{% endtab %}
{% endtabs %}

***

### Keep-Alive / Ping-Pong

The server sends a WebSocket Ping frame every 25 seconds. Your client **must** respond with a Pong frame within 30 seconds, or the connection will be closed.

* **Browser (JavaScript)**: Handled automatically, no code needed.
* **Python (`websockets`)**: Handled automatically, the library responds to pings by default.
* **Rust (`tokio-tungstenite`)**: Pong responses are queued automatically but only flushed when you **read** from the stream. If your code only sends (fire-and-forget), you must spawn a separate task to read from the stream so pongs get flushed. Refer to the Rust example.

If you're using a library that does **not** auto-respond to pings, you must manually send a Pong frame when you receive a Ping. Consult your library's documentation.

***

### Server Limits

<table><thead><tr><th width="260.53125">Parameter</th><th width="167.9921875">Value</th><th>Description</th></tr></thead><tbody><tr><td>Max transaction size</td><td>1232 bytes</td><td>Standard Solana transaction size limit</td></tr><tr><td>Pong timeout</td><td>30 s</td><td>Server sends a ping every 25s; connection closed if no pong received within 30s</td></tr><tr><td>Idle timeout</td><td>300 s</td><td>Connection closed if no transaction sent within 5 minutes</td></tr><tr><td>Handshake timeout</td><td>5 s</td><td>WebSocket upgrade must complete within this window</td></tr><tr><td>Max connections per API key</td><td>5000</td><td>Total concurrent connections allowed per API key</td></tr><tr><td>Max connections per IP</td><td>2</td><td>Total concurrent connections allowed per client IP</td></tr></tbody></table>

***

### Error Handling

The server may close your connection for these reasons:

<table><thead><tr><th width="196.39453125">Reason</th><th>When It Happens</th></tr></thead><tbody><tr><td>Missing API key</td><td>No <code>api_key</code> query parameter in the URL</td></tr><tr><td>Unknown API key</td><td>API key not recognized or inactive</td></tr><tr><td>Connection limit</td><td>Too many connections for this API key or IP</td></tr><tr><td>Pong timeout</td><td>Client stopped responding to ping frames (dead connection)</td></tr><tr><td>Idle timeout</td><td>No transaction sent within the idle timeout window</td></tr><tr><td>Unknown path</td><td>Requested path is not <code>/iris</code> or <code>/mev-protect</code> (rejected during handshake with 404)</td></tr></tbody></table>
