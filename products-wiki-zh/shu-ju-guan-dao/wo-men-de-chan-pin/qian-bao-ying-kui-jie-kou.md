---
description: T该 API 提供评估钱包交易表现的关键指标。
---

# 钱包盈亏接口

**钱包盈亏接口**可在自定义时间范围内返回指定钱包的详细盈亏（PnL）洞察，聚合指标包括：

* **交易量（Trade Volume）**
* **交易次数（Trade Count）**
* **胜率（Win Rate）**
* **总体盈亏（Overall PnL）**

**使用场景（Use Case）**\
适用于跟踪与分析单个钱包的交易表现——用于个人投资监控、研究或对标分析。

**Endpoint**\
`GET /api/v1/dataset/wallet-pnl`

**描述**\
在选定时间窗口内，返回指定钱包的详细盈亏指标：交易量、交易次数、胜率与总体 PnL，帮助全面评估该钱包的交易表现。

***

**查询参数（Query Parameters）**

* **`time`**：用于计算 PnL 的时间窗口。支持值：
  * `2h`：2 小时
  * `3h`：3 小时
  * `5h`：5 小时
  * `10h`：10 小时
  * `24h`：24 小时
* **`wallet`**（_必填_）：要检索 PnL 数据的钱包地址（公钥）。

***

**示例请求**

```
GET /api/v1/dataset/wallet-pnl?time=24h&wallet=F18ibDvGaj4Yjzk1UfWCk5jE9zoy1JYkXNTCBwPuZpyf HTTP/1.1
Authorization: Bearer YOUR_API_KEY
```

该请求返回最近 24 小时内该钱包的 PnL 指标。

**响应格式**

API 返回一个 JSON 对象，包含目标钱包的关键绩效指标。

***

**示例响应**

```json
{
  "wallet": "F18ibDvGaj4Yjzk1UfWCk5jE9zoy1JYkXNTCBwPuZpyf",
  "trade_volume": 12500.75,
  "trade_count": 98,
  "win_rate": 45,
  "pnl_overview": 650.32
}
```

***

**字段说明**

* **`wallet`**：请求中提供的钱包唯一地址。
* **`trade_volume`**：在指定时间窗口内该钱包执行的总交易额（USD）。
* **`trade_count`**：在指定时间窗口内的交易笔数。
* **`win_rate`**：盈利交易的百分比。计算方式：\
  `（盈利笔数 / 卖出金额非零的总交易笔数）× 100`
* **`pnl_overview`**：该时间段内的钱包净盈亏（PnL）。
  * _正数表示盈利，负数表示亏损。_

***

**应用场景（Use Cases）**

* **个人表现监控：**&#x8BC4;估钱包的长期交易表现，发现优势与改进点。
* **投资者尽调：**&#x5728;投资、合作或镜像交易前，评估目标钱包的历史盈利能力。
* **交易策略分析：**&#x57FA;于真实表现的数据，在钱包层面优化或验证交易策略。
