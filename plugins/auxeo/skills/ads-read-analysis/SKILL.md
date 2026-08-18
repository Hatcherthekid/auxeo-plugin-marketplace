---
name: ads-read-analysis
description: 使用 Ads Read MCP 查询、核对并诊断 Meta、Google Ads、TikTok Ads 与 AppsFlyer 投放；适用于事实问数、配置状态、成本或跑量变化、漏斗、素材、回传、政策风险，以及基于证据的行动候选。
---

# Ads · 分析与决策

这是统一的只读分析入口。分类、分析和决策是同一证据链的内部阶段，不拆成三套 Skill：

- 分类：明确对象、grain、时间范围、控制权和问题类别。
- 分析：提出可区分的候选原因，核对支持证据、反证和缺失证据。
- 决策：根据证据强度、影响、风险和审批边界形成 Action Candidate，不直接执行平台写操作。

跨渠道 canonical contract 是 `docs/03_analysis_decision/ADS_CONFIGURATION_INTERPRETATION_AND_TREE_ROUTING_V1.md`。旧
`ADS_CLASSIFICATION_TREE_V1.md`、`ADS_ANALYSIS_TREE_V1.md` 和 `ADS_STRATEGY_TREE_V1.md` 只保留 Meta compatibility 语义，不能覆盖共享合同或直接充当 TikTok / Google 策略。
Phase 5 字段语义遵循 `Fact Evidence -> Derived Feature -> Analysis Observation -> Decision Candidate`；动态
Evidence View 只组装 typed refs，不把事实、特征、规则标签和动作混装。

把 Ads Read MCP 当作真实广告数据能力层，把任务理解、证据选择、继续查询和最终表达留给 Agent。不要把本 Skill 实现成固定路由器、旧 dynamic workflow 或黑盒 Agent。

正式入口是 `auxeo_ads_read` MCP Server 暴露的只读 Tool：`ads_catalog_search`、`ads_catalog_describe`、
`ads_data_health`、`ads_data_query`、`ads_workspace_analyze`、`ads_knowledge_search`、`ads_context_lookup`。
如果 Tool 采用懒加载而暂未显示，应继续按 server 名或 Tool 名检索；不要回退到旧 runtime、旧 gateway、
`AdsCoreClient`、直连 API、本地数据库或脚本冒充 MCP 证据。

## 工作原则

1. 先理解用户真正要判断什么，不先把自然语言塞进预设流程。
2. source、字段或指标不确定时，用 Catalog/Describe 获取正式 definition evidence，不凭记忆猜表、字段或公式；回答保留定义版本、digest 和 source refs。
3. 问题依赖 App 埋点、release/apply 等业务事件含义时，先读取该 App 与业务日期的 Semantic Profile，再查询事实。没有 `published + ready` binding 时只报告已知数据和缺失语义，不自行绑定事件。
4. 平台术语用 Knowledge 解释；当前对象的预算 owner、出价或状态仍用 time-safe Config 证明，不能把通用知识当账户事实。
5. 涉及“最近、今天、昨天、当前”时，用 Data Health 确认数据最新日期和覆盖范围。
6. 用 Data Query 获取事实；根据 partial、空结果和 repair hints 修正 QuerySpec。
7. 只有已有 evidence 需要聚合、排序、对比或派生时才用 Workspace Analyze；简单问题不跑完整工具链。
8. 信息增量不足时换 source、schema、grain、filter 或方法，不重复无效调用。
9. 组织目标、SOP、账户结构或偏好用 Context Lookup；它不能覆盖 metric、source、permission 或 approval。
10. 当前媒体政策可用 Web Search 核对，但网页结论不能覆盖内部账户证据。
11. Tool 确实不可用时说明能力入口不可用，不声称旧代码调用等同于 MCP 调用。

TikTok Report/Entity 必须显式指定 Campaign、AdGroup 或 Ad 的 `data_level`，三层独立解释、禁止跨层求和。平台转化不能冒充 AppsFlyer 经营漏斗。Entity source 不返回 Asset 行；Smart+ 素材关系与 shared Ad envelope 必须查询独立 Asset source，跨 Asset 不可加，也不能称为素材归因。partial/blocked 必须保留 limitation / cannot_judge。

### TikTok 配置、版位与变化证据路由

TikTok 成本或配置变化诊断必须按问题选择最小证据源，并把四类证据结合解释，不能互相替代：

1. `tiktok_config_status_hierarchy`：回答自动/手动版位、自动/手动出价、出价金额、Campaign/AdGroup
   预算及其 owner（CBO/ABO）、状态和层级配置。`placement_type/placements` 只说明允许投放的配置，不证明实际花费版位。
2. `tiktok_placement_af_cost`：回答 TikTok、Pangle、TopBuzz 等实际 placement 的 spend、impressions、clicks、
   CPM/CPC/CTR 与 AppsFlyer installs/CPI/CPS/漏斗。必须保留 `af_with_media / media_only / af_only / placement_unmapped`。
3. `tiktok_entity_temporal_evidence`：在同一对象与合法 `data_level` 上结合日级 performance 和 time-safe Config，
   用于按 package、Campaign、新老、结构、版位/出价配置比较；不把当前配置倒灌成历史配置事实。
4. `tiktok_config_change_events`：回答小时 observation 之间哪些受治理配置字段被检测为变化。`occurred_at_utc` 是检测时间，
   不是 TikTok 平台真实修改时间；零事件且 observation 完整、分页完备、存在可比前序时表示“未检测到变化”，不是缺数；
   对象从后续 inventory 消失不等于 DELETE，只有显式 inactive/delete 状态可证明停投或删除变化。

诊断 Pangle 时，先用实际 placement source 判断是否发生 delivery 及其 CPI/CPS，再用 Config 解释为何具备该投放可能；
不能从自动版位或 `placement_type` 推导 Pangle 花费。诊断“自动出价但未设置金额是否更贵”时，必须在相同 App、层级、
日期与可比结构中同时比较 Config、实际 delivery 和 AF 经营成本；缺少 bid amount 可能是策略语义，不等于数据漏采或原因已证实。

## 诊断与决策

按信息增量选择节点，不强制完整跑树：

1. 明确用户要判断的问题、对象、范围和数据层级，并区分 source Fact、已计算 Feature 与规则 Observation。
2. 选择最能区分候选原因的最小 probe，同时查支持、反证和缺失证据。
3. 查询后重新评估候选原因：保留被支持的，降低或排除有反证的，标记待验证项。
4. 输出有 evidence ref 的 Action Candidate、前提、风险和审批边界；证据不足则明确 cannot_judge。

需要持久化单对象分析证据时，使用 `object_analysis_evidence_v2`：先声明规范化 `analysis_question_key`，再只提供
当前问题需要的 1..N 个 Evidence View。View 可承担 Evidence Gate、投放结构、经营观察、时间变化或原因竞争角色，
但不要求角色集合齐全；支持证据、反证和缺失证据必须分开保存。

需要完整诊断质量检查时，按需读取 [diagnosis-quality.md](references/diagnosis-quality.md)。用户只问数值或明细时，不要强制检索知识或执行完整诊断。

- Knowledge 返回的是候选假设，不是事实、分类结果或规则命令。
- 内部账户事实优先于 Domain Knowledge；当前媒体政策另行核对来源和日期。
- 只选择有信息增量的 probe，没有必要把整棵分析树跑完。

### 预算兑现、子对象分发与可比时序变化

- 先从配置确认预算 owner；只有真实预算控制层才计算预算兑现。
- Ad、Material、Asset 没有预算控制权时，只分析探索、分发、集中度、审核和资格状态。
- 多素材结构中只有少数素材获得消耗可能是正常平台选择，不自动等于预算跑不出或素材异常。
- 配置预算与实际花费先形成 Fact，兑现率是 Feature；是否属于未兑现 Observation 只由预算 owner、可比时间、配置/状态和版本化 Rule Profile 决定。业务量缺口、成本/质量与目标完成情况只进入 Issue、优先级和动作上下文，不改变同一组投放事实的 Observation。
- 只有同一对象、同一 grain 的 delivery 时序同时通过稳定基线、负斜率、连续下降、样本量以及状态/审核/结构/预算出价 gates，才允许形成 draft `temporal_decay_candidate_state`。
- 成本、质量、App 漏斗、素材 CTR 与视频响应只形成独立 change / Issue / evidence，不建立或改变时序状态；新素材零消耗或未探索不能建立长期变化结论。

## 复用和停止边界

- 指标公式来自 Catalog/Describe 和 canonical metric semantic layer，不在 Skill 或 LLM 中重算。
- Report 追问优先消费父 ReportArtifact 和 follow-up anchor；报告生成由报告 Skill 负责。
- 预算兑现、子对象分发和可比时序变化属于分析；当前已有 47 个 Feature Definition、41 个公式、12 个 draft Rule
  Profile、preview evaluator、5 类 draft Cause preview、3 个 operational recipe source slice 和 Phase 5E shadow，
  但 0 个 Profile 获批，尚不发布正式 Observation 或主因排序。预算/出价动作、素材淘汰、复制重建和 Campaign 替换属于后置决策，仍是 `strategy_pending_user_design`。本机旧 `ads-budget-decision` 和历史
  `stable_analysis_v1` 只能作为 compatibility evidence，不能作为跨渠道策略 authority。
- AnalysisFeatureBundle、RuleProfile、preview evaluator 和 approval-gated Observation builder 已实现；没有 approved
  Profile 与 governed Observation artifact 时，不得让 LLM 把 draft temporal preview 或预算状态冒充正式分析；当前 0 个 Profile 获批，因此 Report 不得发布时序衰退结论。
- `partial` 表示证据可能不完整；空结果先排查 scope、source、freshness 和真实无投放，不能自动当成零或完整 cannot_judge。
- 不从名称猜配置、App 或归属；只有时间顺序、机制、对照和反证排除共同支持时才提高因果措辞。
- Domain Pack 不定义正式指标；`reviewed_at` 也不代表账户数据或平台政策的新鲜度。
- 不执行广告平台写操作、飞书正式发送、生产数据库写入或知识晋升。
- 用户反馈与动作结果先进入 candidate / outcome 记录，不能在当前任务直接改全局规则。

## 回答方式

先直接回答，再给关键证据、口径和时间范围、解释及必要限制。建议必须包含对象、证据、动作、优先级和风险前提，并区分已确认、合理推测与待验证。
