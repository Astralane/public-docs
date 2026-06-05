---
description: 用于订阅实时代币价格的 WebSocket API
---

# Websocket - 代币价格

该 WebSocket API 按代币维度提供实时价格更新。客户端可订阅指定代币，持续接收价格变动推送。

### 1. 连接流程

建立 WebSocket 连接：`wss://ws.price.astralane.io/price-by-token`

* 在请求头中携带 `x-api-key`（你的 API Key）
* 发送订阅请求，包含希望订阅的代币列表
* 监听与订阅房间类型对应的事件，接收实时价格更新

### 2. 示例代码

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
  
  // 60 秒后取消订阅
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

### 2.1 响应示例

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

### **3.** 断线重连

若发生断线，客户端需自动重连并重新订阅，确保持续接收不间断的实时更新。5
