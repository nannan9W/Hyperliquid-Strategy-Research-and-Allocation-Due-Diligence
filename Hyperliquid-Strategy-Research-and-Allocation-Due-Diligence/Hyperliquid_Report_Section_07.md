# 7. Tail Risk & Operational Risk

Tail losses arise when hedging, collateral and liquidity protections fail together. Section 5's −0.82% HLP drawdown spans sparse valuation observations and does not bound intraday losses. Section 6's volume-based exit estimates likewise leave order rejection, outages and forced deleveraging unmodeled.

**Margin and hedge failure.** Liquidations depend on maintenance-margin requirements and mark prices, which can differ from executable quotes. A rally can weaken a short-perpetual account while the offsetting spot gain remains unavailable as collateral. Recognition depends on account mode, eligible assets and transfer availability; the modeled 20% cash reserve cannot automatically be counted as immediately usable margin. [Liquidations](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/liquidations), [account modes](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/account-abstraction-modes.md)

Auto-deleveraging can forcibly close profitable opposing positions. For long-spot/short-perpetual carry, this implies residual long exposure if the short is reduced without a matching spot sale. Stress monitoring must therefore track remaining hedge quantities after each event, rather than assume neutrality persists. [ADL mechanism](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/auto-deleveraging)

**Basis, collateral and shared infrastructure.** Funding reversal, adverse spot–perpetual basis changes, UBTC/UETH redemption impairment and USDC depegging create distinct loss channels. Unit facilitates transfers between Hyperliquid and native blockchains; custody, signing controls and redemption continuity remain diligence requirements. [Unit documentation](https://docs.hyperunit.xyz/) Carry and HLP also share Hyperliquid execution and settlement dependencies. Strategy diversification therefore leaves common venue, validator and software risks, while external hedges introduce additional counterparty and transfer dependencies.

**HLP and vault liquidity.** HLP's documented activities include market making, liquidations, USDC supply to Earn and trading-fee receipts. Liquidation inventory and lending exposure require stress assessment beyond historical beta. Its current withdrawal lock lasts four days from the most recent deposit. Other vaults require individually verified withdrawal and fee terms. Withdrawal eligibility, position liquidation and receipt of externally usable cash are separate milestones. [HLP documentation](https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/vaults/protocol-vaults)

**Stress design.** Required scenarios combine rallies or sell-offs with adverse basis, persistent negative funding, reduced depth and delayed transfers. Outputs should include loss relative to all capital, minimum maintenance-margin headroom, residual delta and cash available within 24/72 hours. Reverse stress testing should identify the smallest shock that breaches the mandate's loss budget or collateral requirement. Scenario severities are assumptions; complete execution and liquidation paths remain unverified. Neither a quiet sample nor sparse vault observations establishes reliable tail-event probabilities.

**Operational readiness.** Manager diligence should verify key custody, trading and withdrawal permissions, independent reconciliation, data freshness, API failover, tested cancellation timers and incident ownership. Cancellation removes orders but does not close positions. [Cancellation interface](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/exchange-endpoint#schedule-cancel-dead-mans-switch) Breaches should halt new risk, trigger balance and fill reconciliation, and lead to hedge repair or controlled reduction with documented escalation. Strategy-specific limits require calibration to collateral availability and stressed exits. Allocation approval remains conditional on evidence that these controls operate as described.

*Mechanisms were checked against documentation on 14 September 2026. Rules without verified effective dates inform prospective diligence, not retrospective attribution through the August cutoff.*

---

**作者工作说明：以下内容不纳入英文正文词数。**

本章回答的是：哪些故障会打破已观察到的收益模式，以及管理人需要提供哪些证据，才能进入配置评估。这里没有把拟定的压力测试写成已经完成的实证，也没有用历史最大回撤推断未来损失上限。

**建议配套输出：一张风险与证据表。**

| 风险 | 需要记录或核验的指标 | 取数或证据入口 | 对研究判断的影响 |
|---|---|---|---|
| 保证金与清算 | 可用抵押权益、维持保证金要求、Mark 价格、转账限制、最小保证金余量 | 官方 account modes / margining / liquidations；按账户模式读取账户状态，并核对管理人抵押账簿 | 现金缓冲只有在符合抵押资格且及时可用时，才可计入清算保护 |
| ADL 与单腿成交 | 事件前后现货数量、永续数量、剩余方向敞口、恢复对冲耗时 | 官方成交与账户事件；管理人订单日志；ADL 原因须通过事件证据确认 | 盈利的永续腿仍可能被减仓；仓位减少后需重新计算剩余风险 |
| 资金费与基差 | 小时现金流、两腿可执行价差、持续负资金费时长 | 官方 fundingHistory；同步现货和永续盘口；当前 K 线只支持价格代理 | 资金费恶化与退出不利可能同时发生，需要联合情景 |
| 代币、USDC 与跨链依赖 | UBTC/UETH 赎回路径、托管及签名控制、独立 USDC/USD 报价、转账可用性 | Unit 官方资料与审计/管理人证据；外部美元报价；跨链到账记录 | 即使基础资产价格敞口对冲，也不能消除代币兑付、计价与结算风险 |
| HLP / 金库退出 | 最近一次存入、锁定期、提取资格、实际提取和外部到账时间 | 官方 HLP 说明、vaultDetails、存取款账簿及对应转出记录 | 锁定结束仅代表一个时间条件，不能自动等同于现金已可用于补保证金 |
| 系统与管理人运营 | 数据延迟、订单拒绝、撤单确认、权限变更、对账差异、事故响应责任人 | API / WebSocket 与订单日志、管理人演练记录、脱敏权限清单、事故报告 | 文档中存在功能不等于管理人已正确配置或能在故障中执行 |

表内属于研究核验要求，不表示已获得管理人内部材料。管理人提供的权限证据应脱敏，不需要私钥或其他交易凭证。

**关于图表和截图。**

本章保留上述风险表即可。官方规则页面可作为附录证据截图，截图同时注明抓取时间；不把规则截图当作历史成交或风险暴露数据。入口为正文中的 Liquidations、ADL、Account modes、HLP documentation，以及 [Margining](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/margining)。

若后续增加一张压力损失图，建议横轴为“恢复对冲或调拨资金的延迟”，纵轴为“损失 / 全部配置资本”，分不同价格和基差冲击情景。制作前先核定账户模式、抵押资产资格、保证金阶梯、同步两腿价格及执行规则。当前 4 小时 K 线与假设冲击成本，只足以制作明确标注的条件情景图，不能复原真实清算路径。

**压力测试工作表应保存的字段。**

`scenario_id`、`capital_usdc`、`account_mode`、`spot_quantity`、`perp_quantity`、`eligible_collateral`、`maintenance_margin_required`、`price_shock`、`basis_shock`、`funding_path`、`depth_assumption`、`transfer_delay_hours`、`residual_delta`、`loss_over_total_capital`、`cash_available_24h`、`cash_available_72h`、`evidence_status`。

其中价格冲击、基差冲击和转账延迟可以先作为人为设定的情景参数；损失、余量和可用现金必须由相应模型算出。尚未计算时保留为空，不能填入估计值后称为观察结果。保证金余量应按所使用账户模式的真实公式计算，而不是统一用“抵押金额 / 名义本金”代替。

**证据分级。**

- 已核实事实：本次查阅时官方文档记载的 Mark 清算、ADL、账户模式、HLP 当前四天锁定和订单定时撤销机制。
- 观察推断：ADL 减少空头而现货不变时会留下多头敞口；不同策略共同依赖同一交易与结算系统。
- 模型估计：前文条件收益与成交量退出代理；新的联合压力损失尚未计算。
- 待尽调事项：历史机制生效时间、实际账户权限、代币兑付保障、真实订单执行、日内抵押调拨能力、故障演练结果与管理人未披露账户。

HLP 四天锁定按当前 protocol-vaults 文档表述：从最近一次存入开始计算。官方 legacy 存款人页面另外记载了用户金库的一天锁定，但应先核对具体金库类型、适用日期和版本，不能用 legacy 规则覆盖所有当前或历史金库。

本章机制资料核验日为 2026-09-14 UTC，研究数据截止日仍为 2026-08-31 UTC。没有已核实的历史生效日期时，当前机制只用于后续尽调要求，不被写成整个回测期都已生效的事实。

Messari 适合辅助建立风险事件时间线：分别记录事件发生时间、报道时间、受影响市场与官方原始证据。行情看板或报道没有显示故障，不代表故障未发生。本章具体机制以官方原始说明为依据，不需要 Dune 取数。
