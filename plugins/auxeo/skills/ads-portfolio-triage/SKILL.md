---
name: ads-portfolio-triage
description: 解释跨 App、Meta、Google Ads、TikTok 及 Campaign、Adgroup/Adset、Ad、Material 的受治理日常扫描结果，筛出值得继续分析的对象。用于广告数量很大、需要决定今天先看什么、哪些问题只观察、哪些对象需要下钻，或需要避免从每条 Ad 开始做无用功时。
---

# 广告组合分诊

只解释 `paid_growth_daily_scan_v1` 的 readiness、Portfolio、App 漏斗范围、贡献排序和 bounded observations。不要重新查询全盘数据，也不要在 Skill 中计算指标或筛选阈值。

实际日常入口和父 Report 绑定见
`docs/03_analysis_decision/PAID_GROWTH_DAILY_REVIEW_ENTRYPOINTS_V1.md`；本 Skill 只在该入口已经产生或召回
Daily Scan 后解释分诊结果。

## 工作流

1. 核对 organization、business date、timezone、currency policy、cohort 和 input snapshot。
2. 如果现有 Artifact 的 scope、freshness 或 signature 不匹配，拒绝复用并请求同一 builder 补最小缺口。
3. 先读 data readiness 和 conservation；blocked 时停止，partial 时保留 limitation。
4. 按 App → Channel/Account/Agency → Campaign 阅读贡献观察，识别业务影响集中在哪个范围。
5. 使用 App scope classification 区分跨渠道 App 异常、单渠道异常、单对象异常和 cannot-judge。
6. 只把有治理证据的对象交给 `ads-read-analysis` 或对象证据产品继续下钻。
7. 达到 TaskProfile 的候选数或下钻限制后返回 partial，不自行扩大范围。

## 不做的事情

- 不把 spend 排名等同于“应该优化”的策略排名。
- 不从对象名称猜 App、配置、生命周期或归属。
- 不把 TikTok Smart+ 多 Material 的平台表现相加成素材归因。
- 不在 Ad/Material/Asset 粒度计算预算兑现；子对象零消耗只作为分发/探索观察。
- 不把少数素材承接主要消耗自动标成异常，也不把从未获得分发的素材标成衰退。
- 不把预算 owner 未花完自动标成坏事；必须同时检查业务量缺口、成本/质量和目标完成情况。
- 不用广告平台转化替代 AppsFlyer 经营漏斗。
- 没有批准的 selection policy 时，不生成 `CandidateSelection`。
- 不直接决定加减预算、改出价、停素材、复制重建或更换 Campaign。

## 输出

输出三组：

1. `现在必须看`：数据 Gate 通过且业务影响明确的 bounded observations。
2. `先观察或补证`：partial、样本不足、grain 不足或存在反证的对象。
3. `停止下钻`：已解释主要影响、责任域转向 App/归因/业务，或继续下钻无信息增量的对象。

每项必须带 subject ref、rank、observed metrics、evidence refs、limitation 和下一能力；不得把观察包装成已批准动作。
