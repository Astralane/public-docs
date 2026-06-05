---
description: 面向交易团队与数据驱动型开发者的核心能力概览。
icon: chart-mixed
---

# 我们的产品

**🔌 预构建高性能 API（亚秒级延迟）**

为高要求的交易流程提供超低延迟的链上数据流。

* **Dex Pools API**\
  来自主流 DEX（如 Orca、Raydium）的实时流动池更新。
* **OHLCV API**\
  按可配置时间粒度提供交易对的 K 线数据。
* **Token Price & Volume API**\
  实时代币价格与成交量，支持 WebSocket 实时推送。
* **Pump Fun API**\
  针对 Pump.fun 合约的结构化事件解析，便于快速集成与消费。
* **Profitable Wallets API**\
  按不同时间周期基于收益（PnL）排名的高收益钱包发现。
* **Wallet PnL API**\
  返回钱包级交易指标：PnL、胜率、成交笔数与成交量等。
* **Top 100 Token Holders API**\
  基于钱包余额的任意 SPL 代币 Top 100 持币地址列表（最新快照）。
* **Token Price API**\
  实时获取 SOL 与任意 SPL 代币价格。
* **Token Price（历史）API**\
  按可配置区间获取 SOL 与 SPL 代币的历史价格数据。

***

**⚙️ 自定义数据索引管道**\
基于模块化组件构建并配置专属索引基础设施。

* **Custom Program Indexer**\
  构建定制化的程序级索引器，可选接入外部 API 做数据增强。
* **Transaction Indexer**\
  摄取并索引原始 Solana 交易到热存储，相较标准 RPC 提供更快且更具性价比的查询能力。
* **Custom Account Indexes**\
  高速批量账户索引，绕开频繁调用 `getProgramAccounts` 与 `getMultipleAccounts` 的性能瓶颈与限制。

***

#### 📊 账户级索引（Account Indexer）

覆盖 Solana 网络的全面账户数据索引。

* **Historical Portfolio Values**\
  按日/按月/按年（EOD/EOM/EOY）追踪用户资产组合价值变化。
* **Trade History**\
  针对每个账户的 DEX 级别兑换记录，支持强大的过滤条件。
* **PnL Data**\
  已实现与未实现盈亏指标，支持按日与累计视角统计。
* **Related Wallets**\
  发现与目标账户高频交互的优质关联钱包。

***

#### **📦**  历史数据批量导出（Bulk Dumps）

绕开 API 速率与轮询限制，按需获取可用于分析与机器学习的完整历史数据集。

* **Bulk Portfolio Indexer**\
  为指定钱包提供完整的历史余额与交易记录。
* **DEX Trades Archive**\
  按交易池或代币维度提供历史 DEX 交易数据，支持 CSV 或 Parquet 格式打包分发。

