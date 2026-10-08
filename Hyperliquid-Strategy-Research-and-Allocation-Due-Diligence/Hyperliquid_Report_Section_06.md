# 6. Execution Costs & Capacity

Capacity is the capital allocation that meets return, exit-time and collateral constraints under a specified execution policy. Section 4’s 100,000-USDC replay does not establish that capacity: its assumed execution charge is independent of order size, and 41 of the rule’s 50 nonterminal exits follow the turnover screen. More capital can change both trading costs and the strategy’s participation decisions.

**The completed liquidity diagnostic.** At each daily decision, a separate screen sizes each asset’s matched position at 20% of configured capital using previous-day closing prices. Both legs must fit within 0.1% of their respective previous-day base-unit turnover. Across 365 decisions, both BTC and ETH pass on **91.8%, 33.7% and 1.1%** of days at **25,000, 100,000 and 500,000 USDC**, respectively. Spot is the lower-turnover leg for both assets on every input day. This makes spot the binding leg under this screen, while actual executable depth remains unverified. These frequencies are neither fill probabilities nor measured capacity limits.

**From order size to net return.** A validated capacity curve requires replaying each allocation with the same strategy policy, synchronized spot/perpetual books, and size-dependent execution. Entry buys spot against sells in perpetuals; exit sells spot against perpetual buybacks. Rebalancing costs apply to position changes, while exit tests cover the entire remaining inventory. Partial fills, cancellations, delayed hedges and unfilled quantities must affect subsequent positions and hourly funding cash flows.

Estimate executable prices by walking the appropriate book sides and limiting each slice by available liquidity and participation constraints. Visible depth does not guarantee a fill. Fees, priority payments, financing and operational charges complete the cost account; spreads or slippage already embedded in execution prices must not be deducted again. Larger allocations require recalculating attainable fee tiers from dated schedules and the account’s own rolling turnover. Today’s discounted rates cannot establish historical eligibility. [Official fee schedule](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees)

**Stress exits and capital efficiency.** Retain Section 3’s normal/stress calendar and compare standardized exit orders within matched time strata. Report each leg’s completion time, joint completion time, cost and residual exposure. The slower leg governs joint completion; unfinished exits remain censored and visible. Higher trading volume during stress does not guarantee tighter spreads or available depth.

For this chapter, an **illustrative** capacity gate requires at least **2% net return over the one-year research window**, completion of at least **95% of stress exits within 24 hours**, and no breach of the configured collateral buffer. Returns use all capital, including the 40% perpetual-collateral allocation and 20% liquidity reserve. Transfer delays, unavailable balances and adverse mark-price paths may bind before market depth. The 2% hurdle and exit deadline are research assumptions, not established investor requirements. Section 4’s passive +1.93% estimate already misses that illustrative return gate at 100,000 USDC; no investable capacity is established.

**Priority fees and making markets.** Messari’s July study reports that IOC priority charges absorbed approximately **69%** of visible gross returns in its selected high-fee-market cohorts. Its separate edge analysis uses short-horizon markouts; those findings cannot be applied as a blanket deduction from funding carry. [Messari priority-fee research](https://messari.io/report/hyper-liquid-priority-fees-internalizing-high-frequency-trading-revenue)

Priority costs must follow actual billing: IOC charges depend on filled notional; ALO placement can incur charges without a fill; read priority has separate auction costs. Fee-paying HYPE balances also require funding. [Official priority-fee mechanics](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/priority-fees) For market making, inventory paths, fill probability, adverse-selection markouts and queue competition require order-level evidence. L2 snapshots alone cannot support an exact market-making replay. The capital–net-return curve and stress-exit frontier therefore remain unmeasured until the required execution history is available.

---

# 作者工作说明：不纳入英文成稿字数

本章延续第四章的 BTC/UBTC 与 ETH/UETH 配对、资本分配和样本期。英文正文已写入实际完成的成交量筛选诊断，同时明确区分尚未取得数据的执行容量曲线。没有将看板截图或 K 线成交量改称真实滑点、成交概率或退出时间。

## 已完成的规模诊断

这个诊断是独立的每日事前筛选，不是重新运行第四章策略：固定配置资金 K，在每日 00:00 UTC 只使用前一个完整 UTC 日的收盘价和成交量。各资产的单腿目标数量为：

\[
q_i(K)=\frac{0.2K}{\max(S_{i,d-1}^{close},P_{i,d-1}^{close})}.
\]

两腿同时满足 `q <= 0.001 × previous_day_base_volume` 才通过。前一日输入包括 2025-08-31 预热数据，共形成 365 个决策日。

| 配置资金 K，USDC | BTC 通过天数 | ETH 通过天数 | 两组配对同时通过 | 同时通过比例 |
|---:|---:|---:|---:|---:|
| 25,000 | 365 | 335 | 335 / 365 | 91.8% |
| 100,000 | 302 | 125 | 123 / 365 | 33.7% |
| 500,000 | 47 | 5 | 4 / 365 | 1.1% |
| 1,000,000 | 12 | 3 | 2 / 365 | 0.5% |
| 5,000,000 | 0 | 0 | 0 / 365 | 0.0% |
| 10,000,000 | 0 | 0 | 0 / 365 | 0.0% |

0.1% 是第四章采用的筛选参数，不是交易所承诺的可成交比例。筛选整个目标头寸，也不等于每天都交易整个头寸。实际成交量参与率通常应针对本次订单数量或切片，退出测试则检查剩余全部头寸。

现货在全部输入日的基础币成交量均小于对应永续，这能确定“成交量筛选的限制腿”。是否同样限制真实退出，仍要看卖出现货的买盘、回补永续的卖盘、撤单、延迟、费用和保证金。不能把 33.7% 写成“成交成功率 33.7%”。

可复现数据见 `inputs/prior_day_volume_price_inputs.json`，逐规模结果见 `outputs/size_screen_diagnostic.json`。四个交易腿的官方原始 4 小时响应保存在 `inputs/`。运行 `python3 reproduce_size_screen.py` 可以离线复算。

## 核心图的设计

**图名：Capital Allocation vs Net Return After Execution Costs**

一张组合图即可，不再扩大正文图数：

| 图内位置 | 横轴 | 纵轴或输出 | 固定口径 |
|---|---|---|---|
| 主图 | 配置资本 25k、100k、500k、1m、5m、10m USDC，对数刻度 | 全样本成本后净收益率 | 被动与规则版本分别重跑；展示基准执行与单独标注的压力退出情景，画 2% 研究假设门槛线。 |
| 下方成本图或表 | 同一配置规模 | 正常、压力退出成本的 P50/P95 | 默认除以全部配置资本，以 bps 表示；若另外报告单腿名义金额口径，必须另列分母。 |
| 下方时间图或表 | 同一配置规模 | 各腿及双腿退出时间的 P50/P95、24 小时未退出比例 | 未成交余量保留；不能只对成功退出样本计算一个看似良好的 P95。 |

基准曲线应使用研究期真实发生的正常与压力环境。压力曲线是明确的反事实执行情景，不能把“压力日平均回报”当作全年净收益。若压力平仓替换了原平仓，删除原平仓费用再计入新费用，避免额外收两次完整平仓成本。

每个配置规模必须重算订单、成交、剩余仓位、费用档位、资金费和现金余额。直接放大十万美元的毛 PnL，再随意扣一个与规模有关的 bps 数值，不足以证明容量。资金费路径不随自身交易改变也应作为条件假设，另检验拥挤导致资金费收窄的敏感性。

容量定义可写成：

\[
K^*=\max\left\{K:\ R_{net}(K)\ge h,\quad
\Pr(T_{exit}(K)\le24h\mid stress)\ge95\%,\quad
E_{perp}(t)-MM(t)-B_{required}(t)\ge0\right\}.
\]

这里取 `h=2%`，适用于本次一年研究窗口。`MM` 是按当时有效规则计算的维持保证金；`B_required` 是预先设定的额外缓冲。20% 总资本流动性储备和额外保证金缓冲不是同一个指标。当前无法验证所有条件，所以 `K*` 的状态是 **未建立**，不能记为已测得零容量或十万美元容量。

建议同时展示 h=0%、2%、5% 的敏感性，避免将单一研究门槛误写成通用投资标准。这些是后续容量研究规格，没有被用于挑选本次已完成的回测结果。

## 去哪里取数，截图能做什么

| 数据 | 优先来源 | 必须保留的内容 | 能否仅靠截图完成 |
|---|---|---|---|
| 同步现货与永续盘口 | 官方 `l2Book` / WebSocket；历史已有归档或自己此前留存的数据 | 交易对、服务端与接收时间、买卖方向、每档 `px/sz`、快照年龄、缺失和截断标记 | 不能。单张截图只能展示某时刻盘口，不能计算历史容量或退出时间。 |
| 成交、撤单、部分成交 | 官方节点记录或可核验账户的订单、fills 记录 | 订单 ID、提交/确认/成交时间、数量、手续费币种、取消与拒绝原因 | 不能。需事件序列校验成交概率及双腿先后关系。 |
| 历史费率与真实档位 | 官方费率、当期公告、实际账户费用记录 | 生效日期、现货/永续、maker/taker、14 日加权成交量、实际折扣 | 截图可保存费率证据；今日截图不能证明历史费率。 |
| 保证金及调拨 | 官方当期 margin table、账户模式与余额/转账记录 | Mark、权益、维持保证金、可转余额、到账时间、HYPE 费用储备 | 当前截图只说明当前状态，无法证明压力时段可以调拨。 |
| Priority fees 研究背景 | Messari 的指定报告 | 研究日期、样本范围、测量周期、费用与收益定义 | 可截图解释研究背景；不能拿协议收入图当成本曲线。 |

官方 [L2/API 说明](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/info-endpoint) 显示快照每侧最多 20 档。超过可见深度的数量必须标记未覆盖，不能按最后一档价格无限成交。聚合档位也不能恢复精确队列位置。

当前只读请求示例为：

```json
{"type":"l2Book","coin":"BTC"}
```

其他腿使用 `ETH`、UBTC/USDC 与 UETH/USDC 对应的现货标识。第四章保存的映射为 `@142`、`@151`；新取数时先通过 `spotMeta` 复核。上述请求返回当前盘口，不接受一个历史日期就自动还原过去盘口。

官方历史数据路径示例是 `s3://hyperliquid-archive/market_data/[date]/[hour]/l2Book/[coin].lz4`。官方说明历史上传可能延迟或缺失，并明确不在该 S3 数据集中提供 spot asset data。**不能凭永续归档的存在，就声称 UBTC/UETH 历史盘口也有完整覆盖。** 需要检查具体可得数据；现货缺口可使用此前自行保存的官方推送或经核验的节点重放。若只能现在开始采集，应新增向前观察样本，并标注真实日期，不能回填至 2026-08-31。[官方历史数据说明](https://hyperliquid.gitbook.io/hyperliquid-docs/historical-data)

## 正常和压力退出的计算约定

沿用第三章已保存的日历：BTC 永续日内 `log(high/low)` 处于全样本 P40–P60 的 73 天作为正常组，达到 P90 的 37 天作为压力组。它是事后描述标签，不能作为当日事前交易信号。

对标准化的全额配置退出，固定 BTC/ETH 各 20% 单腿名义金额和同一执行政策；在共同月份、小时分层，等权比较日期，并展示缺失的分层。实际策略的持仓退出可另检验，不与标准化测试混合。

对于单个资产，退出需要卖出现货并买回等量永续。在同一执行起点下，组合完成时间至少受两腿中较晚完成者限制；全组合再取 BTC、ETH 两组的较晚完成时间。每个切片受可见深度、实际成交量参与约束、订单确认和对冲状态限制。重复出现的同一批挂单不能每个快照都重新计为新供应；仅凭聚合快照无法完全辨认时，应采用明示的保守补单假设并标成模型估计。

回补永续必须预留抵押及费用，现货卖出收入只有在实际可用后才能用于调拨。现货升值不能自动保证独立永续账户有足够资金。历史账户模式、分档保证金与强平路径必须按当时规则验证。[保证金说明](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/margining)、[清算说明](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/liquidations)

## Priority fees 的使用方法

Messari 的起点报告为 [Hyperliquid Priority Fees: Internalizing High Frequency Trading Revenue](https://messari.io/report/hyper-liquid-priority-fees-internalizing-high-frequency-trading-revenue)，页面标注 2026-07-29。重点阅读 **Current Adoption of Priority Fees**、**Modeling Write Priority Fees** 和 **Measuring the Latency Stack**。前两部分区分了选定账户的可见收益与 30 秒 markout 机会估计，不应将它们混成整套资金费率策略的实际净收益。

可截图保留研究的样本和图注，用自己的文字说明含义；核心容量图仍需自行计算。该文记载 2026-04-13 启用 priority fees，可作为追查生效时间的起点；历史成本还需匹配机制版本和账户实际使用，不能把当前机制套用全年。

IOC、ALO、read priority 应分账。IOC 以已成交数量计费；ALO 可能对没有成交的挂单收费；read priority 另有拍卖支付。分别从实际对应日志识别费用，核对 HYPE 数量及其支付时估值。第四章“priority fee=0”是策略不购买优先权的假设，不是断言其他参与者不付款。[官方 priority fee 说明](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/priority-fees)

做市研究至少保留订单进入和离开队列、成交或未成交、库存风险、成交后的多时点 markout、重报频率、base fee/rebate、priority cost。ALO 成本不能只摊到成交订单后遗忘未成交订单费用。没有这些记录，本章只评价可见条件与成本预算，不提供声称精确复现的做市回测。

## 图表状态与投稿措辞

已完成：规模筛选表、逐日输入、可复算代码，以及本章正文。

未完成：同一历史区间内同步双腿执行数据、规模相关成交价格、压力退出成本/时间，以及满足全部约束的容量数值。因此本次没有绘制虚构的成本后收益曲线。正文已经使用与该证据状态一致的表述。补齐数据后，可用核心组合图替换方法说明中的部分文字，控制全文在 3,000–4,000 英文词。
