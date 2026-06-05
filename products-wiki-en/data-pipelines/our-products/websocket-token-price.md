---
description: Websocket API for token prices
---

# Websocket - Token price

The WebSocket Token Price API provides real-time updates for token prices by token. Clients can subscribe to specific tokens and receive live price updates.

### 1. Connection Workflow

Establish Websocket connection to the `wss://ws.price.astralane.io/price-by-token`

1. Specify your API key in `x-api-key` header;
2. Send subscription request for desired list of tokens;
3. Listen for events corresponding to the subscribed room type to receive real-time price updates.



### 2. Sample code

```javascript
const WebSocket = require('ws');

const tokens = [
  "So11111111111111111111111111111111111111112", 
  "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"
];

const socket = new WebSocket('wss://ws.price.astralane.io/price-by-token', {
  headers: {
    'x-api-key': 'YOUR-API-KEY'
  }
});

let messageCount = 0;
let unsubscribeTimeout;

const subscribeEvent = {
  event: "subscribe",
  tokens: tokens,
};

const unsubscribeEvent = {
  event: "unsubscribe", 
  tokens: tokens,
};

socket.on('open', function() {
  console.log('Connected to server');
  console.log('Subscribing to tokens:', tokens);

  socket.send(JSON.stringify(subscribeEvent));
  
  // Set timeout to unsubscribe after 60 seconds
  unsubscribeTimeout = setTimeout(() => {
    console.log('Unsubscribing after 60 seconds...');
    socket.send(JSON.stringify(unsubscribeEvent));
    socket.close();
  }, 60000);
});

socket.on('message', function(data) {
  messageCount++;
  console.log(`Received message #${messageCount}`);
  
  try {
    const parsedData = JSON.parse(data.toString());
    console.log('Parsed data:', parsedData);
  } catch (e) {
    console.log('Raw data (not JSON):', data.toString());
  }
});

socket.on('error', function(error) {
  console.error('WebSocket error:', error);
  clearTimeout(unsubscribeTimeout);
});

socket.on('close', function(code, reason) {
  console.log(`Connection closed: [${code}] ${reason}`);
  console.log(`Total messages received: ${messageCount}`);
  clearTimeout(unsubscribeTimeout);
});


```

### 2.1 Response example

```json
{
    "timestamp": "1741771359",
    "decimals": 9,
    "name": "Wrapped SOL",
    "symbol": "SOL",
    "uri": "https://raw.githubusercontent.com/solana-labs/token-list/main/assets/mainnet/So11111111111111111111111111111111111111112/logo.png",
    "supply": 509384498.1158462,
    "price_in_sol": 1,
    "price_in_usd": 124.3285941413753,
    "token": "So11111111111111111111111111111111111111112"
}
```

### **3. Disconnection**

In the event of a disconnection, the client must automatically reconnect and resubscribe to continue receiving uninterrupted real-time updates.
