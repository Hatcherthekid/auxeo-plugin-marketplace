---
name: ads-reporting
description: 设计、生成和追问广告报告；报告定义、字段、章节、交付配置和可用目的地由服务端动态声明。
---

# Ads · 报告

## 入口

新任务先调用 `ads_capability_context`，确认当前授权范围、报告能力和 server guidance。只有 capability context 暴露正式 Report 生成能力时，才调用它返回的当前服务端入口。不要把固定 App、账户或飞书标识写进报告入口。

## 编排

1. 明确受众、日期、时区、范围、指标口径、比较方式、重点、cadence 与展示偏好；指标口径使用 `cohort` 或 `actual`，这些需求不等于服务端当前都已支持。
2. 创建新报告时，先把选定的 `fact_semantics` 传给 Catalog 选择 source，再用 Describe 核对字段；把同一口径传给 Query，并用 Workspace SQL 或受限 Python 把各 grain 物化为当前成员可读的 Evidence Artifact；不要让 Report renderer 重新取数或计算指标。
3. 从 `report.user_defined.v1` capability card 读取 `contract_ref`，再读取该 MCP Resource 取得当前服务端的完整 ReportSpec JSON Schema、composition envelope、digest 和示例；不要猜嵌套字段。无法由已发布字段或 Workspace 表达时返回 unsupported，不编造配置。
4. 把 `mode=preview|generate`、`primary_window`、`spec`、`input_refs` 和可选 `feishu_projection` 放入 `composition`；`report_id` 对用户定义报告可省略并由服务端按 spec 生成。先创建真实数据 preview，用户确认后复用同一 ReportSpec、换成最新 Evidence Artifact 生成正式 ReportArtifact。
5. 用户定义 Report 完全继承 Evidence Artifact 的 principal/binding scope，不要求 Report 权限、App 权限、固定账户或单一 `resource_ref`。只有兼容的已注册 Google 报告继续使用 opaque `resource_ref`；`app_ref` 仍只是查询 facet。
6. 用户定义报告优先使用 `composition.primary_window`，连续范围最多 92 天；已注册兼容报告继续使用 `report_config.primary_window` 或旧 `report_date`。滚动周 section 必须声明源行的 `start_field/end_field`，编译器会过滤超窗行。`include_png_long_image` 生成长图；`composition.feishu_projection=true` 还要求长图，并且只返回不含收件人或 token 的 ProjectionPlan Artifact。
7. 生成前校验 source、metric、grain、freshness、稳定 `ads.read` 与数据覆盖；Report 本身没有独立权限。追问优先复用父 artifact 与 anchor，必要时再补证。
8. 区分 preview、已生成 Artifact、已发布版本、ProjectionPlan 与 delivery receipt；只有 receipt 能证明已发送。
9. 同一 section 只能使用一种 fact semantics；需要双口径时拆成独立 section，并分别标注日期语义，禁止合并求和。

## 边界

- 文案不能覆盖事实、不确定性或正式指标定义。
- 没有可用 destination、配置权限或审批时，只返回草案与缺口。
- 正式 Report 生成入口只生成受权限约束的 Report/Artifact，不代表已配置排期、发布或交付。
- 不直接发送飞书/IM，不修改广告平台。
