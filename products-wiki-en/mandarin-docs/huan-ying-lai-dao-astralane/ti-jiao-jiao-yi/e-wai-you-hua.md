# 额外优化

**持久连接**

“Keep-Alive”（亦称持久连接）是 HTTP 或 TCP 等网络协议中用于在客户端与服务器之间保持长连接的功能。启用该功能后，可以处理多个请求 / 响应，而无需每次都重新建立连接。

**目的**

主要目标是通过避免频繁打开和关闭连接来降低延迟和开销。这对于高性能系统或处理大量短时间请求的场景尤其有用。



关键特点

* **连接重用**：同一 TCP 连接可用于多个请求，消除频繁建立和关闭连接的成本。
* **延迟减少**：无需进行 TCP 握手所需的往返时间，从而提升整体响应速度。
* **吞吐量提升**：保持连接通道开放，允许更多请求通过同一连接高效处理。
* **资源消耗降低**：减少客户端和服务器在连接频繁切换上的 CPU 和内存使用。
* **心跳或 Ping 机制**：多数实现会定期发送保持活动的信号，避免中间网络设备将连接视为空闲或关闭。

```
async fn send_txn_with_keep_alive() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::builder()
        .pool_idle_timeout(Some(Duration::from_secs(85))) // Keep connections alive for 85
        .build()?;
```

[这是带有示例的完整源代码](https://github.com/Astralane/keep-Alive-Example/blob/main/src/main.rs)
