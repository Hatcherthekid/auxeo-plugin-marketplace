---
name: paid-growth-operations
description: 编排 Meta、Google Ads、TikTok App 买量的日常巡检、专题诊断、周期复盘、动作跟进和 Report 追问。用于用户询问三个渠道今天先处理什么、整体买量为什么变差、如何复盘大量 Campaign/Adgroup/Ad/Material，或需要把既有报告、广告证据、AppsFlyer 漏斗和动作结果合并成统一行动队列时。
---

# 付费用户增长运营

把本 Skill 当作顶层方法编排，不把它当成计算器或巨型分析 Prompt。复用受治理 Artifact 和叶子能力，最后按 App 和 Issue 汇总。

## 选择 TaskProfile

先根据任务选择一个明确 Profile，不按关键词递归加载所有 Skill：

- 日常全盘巡检：`paid_growth_daily_review_v1`
- 单对象分析：`paid_growth_object_analysis_v2`
- App 漏斗范围判断：`paid_growth_app_conversion_diagnosis_v1`
- 已执行动作复盘：`paid_growth_action_followup_v1`
- 日报、周报或 section 追问：`paid_growth_report_followup_v1`

简单问数直接使用 `ads-read-analysis`，不要触发完整复盘。

## 当前运行入口

本地 canonical builder 的日常扫描入口是 `run_paid_growth_daily_scan.py`；已选对象的局部事实入口是
`run_paid_growth_phase5e_shadow.py` 或 `run_paid_growth_analysis_recipe.py`。它们只生成受治理的只读
Analysis Artifact。完整顺序、现有 Meta/Google/TikTok 日报绑定和 Follow-up 规则见
`docs/03_analysis_decision/PAID_GROWTH_DAILY_REVIEW_ENTRYPOINTS_V1.md`。

已有报告时先消费其 artifact 与 anchor；不要为了使用本 Skill 另起一份日报、启用新 route 或重算父报告指标。

## 执行顺序

1. 明确 organization、App、渠道、业务日期、时区、币种政策和 D0/D1。
2. 优先召回完全匹配的 `ReportArtifact` 或 `paid_growth_daily_scan_v1`；检查 scope、freshness、cohort、anchor 和 analysis signature。
3. 没有可复用产物时，只有已经绑定 canonical builder 的本地 Host 才能触发同一 builder 的 lazy build；只具备 MCP 的弱 Host 必须返回 artifact missing，不能在 Prompt、Report renderer 或临时脚本中重做全盘扫描。
4. 使用 `ads-portfolio-triage` 解释 bounded observations，确定是否值得补钻。
5. 只有被选中的对象才使用 `ads-read-analysis` 和 `object_analysis_evidence_v2` 下钻；先声明规范化
   `analysis_question_key`，再只查询有信息增量的 Evidence View，不补齐固定集合。
6. 涉及 App 范围异常时使用 `app-conversion-diagnosis`，先确定责任域和广告动作 Gate。
7. 将同一 stable subject + symptom 的证据合并为一个 Issue；保留支持证据、反证、缺失证据和最新 observation。
8. 只有已有 Action、Outcome 和 due queue 均 ready 时才复盘动作效果；不要把同期改善自动归因于动作。
9. 输出统一 Review Artifact；需要正式报告时交给 Report Kernel 的 shadow/approved adapter，不直接拼接报告正文。

## 证据和停止规则

- 数据完整性、归因、grain、币种和 cohort 先走工程 preflight；缺口进入回补/修复队列，并只阻断受影响的结论，不能成为经营主矛盾或业务主因。
- 证据按问题动态选择：先过 freshness/归因/grain/样本 Gate，再看投放结构、经营观察、时间变化和原因反证。
- 字段按 `Fact Evidence -> Derived Feature -> Analysis Observation -> Decision Candidate` 分层；Evidence View 只是
  读模型。没有版本化 Feature/Rule artifact 时不临时生成正式状态。
- 预算兑现只在真实预算 owner 判断；Ad/Material/Asset 无预算控制权时只分析探索、分发、集中度和状态。
- 子素材无消耗不自动等于预算未兑现、差素材或衰退；预算未兑现也不自动等于负面问题。
- 从上到下发现值得看的对象，从下到上验证候选原因，最后回到 App 业务影响。
- 已解释主要影响、继续下钻无权威 grain、样本不足、责任域不在广告侧，或已达到 Profile 资源上限时停止。
- Host 未执行 tool allowlist、调用次数、超时、候选数量和下钻深度限制时，只能返回 `evidence_only`。

## 当前策略边界

当前已实现 47 个中性 Feature Definition、41 个确定性公式、12 个 draft Rule Profile、只读 preview evaluator、
5 类 draft Cause preview，以及 3 个 operational recipe source slice 和 Phase 5E shadow 接线；但 0 个 Rule Profile
获批，因此仍不发布 governed Observation，也不做主因排序。预算/出价动作、素材淘汰、复制重建和 Campaign
替换属于后置决策能力。没有批准的 selection/decision Profile 时，只呈现 preview、候选原因和限制，不生成强动作或 CandidateSelection。

不执行广告平台写操作，不静默改写父 Report、Decision、Action 或 Issue 历史。

## 输出

按以下顺序输出：

1. 当前结论和数据适用范围。
2. 按优先级排列的 Issue：对象、业务影响、证据、反证、缺口、责任域、广告动作 Gate。
3. 建议的下一步补证或待审批动作；未批准策略只写方向和前提。
4. 已达到的停止条件、cannot-judge 和下一复核时间。
