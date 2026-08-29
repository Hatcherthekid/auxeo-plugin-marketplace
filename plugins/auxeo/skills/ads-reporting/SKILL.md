---
name: ads-reporting
description: 设计、生成和追问广告报告；报告定义、字段、章节、交付配置和可用目的地由服务端动态声明。
---

# Ads · 报告

## 入口

新任务先调用 `ads_capability_context`，确认当前授权范围、报告能力和 server guidance。只有 capability context 暴露正式 Report 生成能力时，才调用它返回的当前服务端入口。不要把固定 App、账户或飞书标识写进报告入口。

若当前 Host 能读取 Auxeo MCP Resources、但没有披露依赖的自定义 Tool，先读取 `ads-contract://host-tool-fallback-v1`，再按其中的 `ads-query://execute/{tool_name}{?arguments}` Resource Template 调用同一只读能力。该入口仅用于 Host 恢复，不扩大权限，也不得绕过正常 Tool。

## 编排

1. 明确受众、日期、时区、范围、指标口径、比较方式、重点、cadence 与展示偏好；指标口径使用 `cohort` 或 `actual`，这些需求不等于服务端当前都已支持。
2. 创建新报告时，先把选定的 `fact_semantics` 传给 Catalog 选择 source，再用 Describe 核对字段；把同一口径传给 Query，并用 Workspace SQL 或受限 Python 处理当前查询结果。普通 Query 不创建长期证据；只有明确保存或正式 Report run 才发布当前成员可读的 `PublishedEvidenceSet`。不要让 Report renderer 重新取数或计算指标。
3. 从 `report.user_defined.v1` capability card 读取 `contract_ref`，再读取该 MCP Resource 取得当前服务端的完整 ReportSpec JSON Schema、composition envelope、digest 和示例；不要猜嵌套字段。无法由已发布字段或 Workspace 表达时返回 unsupported，不编造配置。
4. 把 `mode=preview|generate`、`primary_window`、`spec`、`input_refs` 和可选 `feishu_projection` 放入 `composition`；`report_id` 对用户定义报告可省略并由服务端按 spec 生成。先创建真实数据 preview，用户确认后复用同一 ReportSpec、换成最新 `PublishedEvidenceSet` 生成正式 `ReportRun` 与 `RenderedAsset`。
5. 用户定义 Report 完全继承 `PublishedEvidenceSet` 的 principal/binding scope，不要求 Report 权限、App 权限、固定账户或单一 `resource_ref`。只有兼容的已注册 Google 报告继续使用 opaque `resource_ref`；`app_ref` 仍只是查询 facet。
6. 用户定义报告优先使用 `composition.primary_window`，连续范围最多 92 天；已注册兼容报告继续使用 `report_config.primary_window` 或旧 `report_date`。滚动周 section 必须声明源行的 `start_field/end_field`，编译器会过滤超窗行。`include_png_long_image` 生成长图；`composition.feishu_projection=true` 还要求长图，并且只返回不含收件人或 token 的 ProjectionPlan。
7. 生成前校验 source、metric、grain、freshness、稳定 `ads.read` 与数据覆盖；Report 本身没有独立权限。追问优先复用父 `ReportRun`、`PublishedEvidenceSet` 与 anchor，必要时再补证。
8. 区分 preview、`ReportRun`、`RenderedAsset`、已发布版本、ProjectionPlan 与 delivery receipt；只有 receipt 能证明已发送。
9. 同一 section 只能使用一种 fact semantics；需要双口径时拆成独立 section，并分别标注日期语义，禁止合并求和。
10. 报告合同同时固定 counting semantics：漏斗转化率默认 `unique_users`，事件活跃度默认 `total_events`。累计多日漏斗率使用 install-cohort UV（默认 D0；显式 D1/D3 必须由支持该窗口的 source 返回），actual UV 仅用于单日 actual 或按日趋势，禁止用其 Daily UV Sum 形成累计转化率。Daily/object UV 相加的标题或脚注必须写 `Daily UV Sum / 每日 UV 之和`，并说明跨日重复用户会重复计数、不是区间真正去重 UV。

## 边界

- 文案不能覆盖事实、不确定性或正式指标定义。
- 没有可用 destination、配置权限或审批时，只返回草案与缺口。
- 正式 Report 生成入口只生成受权限约束的 `PublishedEvidenceSet`、`ReportRun` 与 `RenderedAsset`，不代表已配置排期、发布或交付。
- 不直接发送飞书/IM，不修改广告平台。
