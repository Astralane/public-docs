---
icon: bolt-lightning
---

# 附加优化

## Keep-Alive 连接：

Keep-Alive（持久连接）是 HTTP、TCP 等网络协议中的特性，用于在客户端与服务器之间保持一条长连接，以承载多次请求与响应，而非每次交互都新建/关闭连接。

### 目的：

Keep-Alive 的核心目标是降低时延与开销，避免反复的建连与断连。该特性在高性能系统或需处理大量短生命周期请求的场景尤为有效。

### 关键特性

* 连接复用：通过复用同一条 TCP 连接，避免频繁的连接建立与释放。
* 降低延迟：跳过新建连接所需的握手往返，缩短整体响应时间。
* 提升吞吐：维持传输通道处于可用状态，在同一连接上高效处理更多请求。
* 资源友好：减少客户端与服务端的 CPU、内存开销，降低连接抖动。
* 心跳/保活：可配置周期性保活信号，避免连接在空闲期间被中间网络设备回收或关闭。

**示例代码**

```
async fn send_txn_with_keep_alive() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::builder()
        .pool_idle_timeout(Some(Duration::from_secs(85)))  // 空闲连接保活 85 秒
        .build()?;
```

[完整实例](https://github.com/Astralane/keep-Alive-Example/blob/main/src/main.rs)以供参考<br>
