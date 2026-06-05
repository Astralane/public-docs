---
description: 按 PnL 排序的高收益钱包
---

# 高收益钱包

**高收益钱包 API** 基于交易活动产出最具表现的钱包洞察，聚合并计算以下关键指标：

* **交易量（Trade Volume）**
* **交易次数（Trade Count）**
* **胜率（Win Rate）**
* **总体盈亏（PnL）**

上述指标可按用户自定义时间窗口统计，提供细致的钱包绩效视图。

**使用场景：**&#x8BC6;别高绩效交易者、开展竞品/对手盘分析、理解生态内的钱包行为特征。

**查询参数**

* **`time`**（_必填_）：指定统计时间窗口。支持值：
  * `2h`：2 小时
  * `3h`：3 小时
  * `5h`：5 小时
  * `10h`：10 小时
  * `24h`：24 小时
  * `3d`：3 天
  * `1W`：1 周
* **`page`**（_可选_）：分页页码，**默认** 1
* **`limit`**（_可选_）：每页返回的记录条数，**默认** 100

***

**示例请求**

```
GET https://graphql.astralane.io/api/v1/dataset/profitable-wallets?time=24h&page=1&limit=50
```

该请求返回过去 24 小时内最赚钱的前 50 个钱包。

***

**响应格式**

API 返回一个 JSON 数组，每个对象代表一个钱包及其相应绩效指标。



**示例响应**

```json
[
  {
    "wallet": "HmD1XGGXLRnYvq537jmTUuhcRgecZF3NeNoFtnBv5ERX",
    "trade_volume": 15705.95985742,
    "trade_count": 149,
    "win_rate": 40,
    "pnl_overview": 478.16333295
  }
]
```

**字段说明（Field Descriptions）**

* **`wallet`**：钱包唯一标识（通常为公钥/地址）。
* **`trade_volume`**：在所选时间范围内该钱包执行的总交易额（USD）。
* **`trade_count`**：在所选时间窗口内的总交易笔数。
* **`win_rate`**：盈利交易占比（盈利笔数 / 总笔数，百分比）。
* **`pnl_overview`**：该时间段内钱包的净盈亏（PnL）。为正表示盈利，为负表示亏损。

***

**应用场景（Use Cases）**

* **投资者分析（Investor Analysis）**：识别值得关注或镜像的高绩效钱包。
* **市场研究（Market Research）**：洞察生态中头部钱包的交易行为与表现。
* **竞争情报（Competitive Intelligence）**：比较不同钱包的盈利能力，分析策略并发现市场趋势。

***

**分页说明（Pagination）**

为处理大数据量，API 支持分页：

* **`page`**：结果集的页码。
* **`limit`**：每页返回的记录数。
* **每页最大支持的 limit 值：**&#x31;00。

