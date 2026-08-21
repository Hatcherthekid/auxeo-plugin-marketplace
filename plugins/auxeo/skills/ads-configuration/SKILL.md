---
name: ads-configuration
description: 定义和核对广告数据中的实体 mapping、指标/KPI、标签、组织上下文与 source authority，验证配置是否可用；当前只读，不修改平台或发布外部设置。
---

# Ads · 数据定义与匹配

这是分析、监控和报表共同依赖的基础配置入口，回答“数据是谁、口径是什么、怎样匹配、能否作为证据”。广告对象配置属于平台配置核查；监控规则和报表版式分别由对应 Skill 负责。

## 定义与匹配

1. 实体 identity mapping：账户、应用、渠道、Campaign、AdGroup/Ad、AppsFlyer 应用和内部业务对象。
2. 指标与 KPI：用户目标、标签、分组和别名到正式 metric/source/grain 的映射。
3. 组织上下文：业务时区、负责人、账户归属、SOP 和默认分析偏好。
4. 数据链路健康：source authority、同步日期、字段覆盖、AppsFlyer 回传、freshness 和 mapping completeness。

## 工作流程

1. 明确要定义或匹配的对象，以及由分析、监控还是报表消费。
2. 指标定义问题先用 `ads_catalog_search` / `ads_catalog_describe` 读取正式定义 evidence，保留 `definition_digest`、`registry_revision` 和 `source_refs`；不要在 Skill 中抄写或重算公式。
3. App 埋点或 source event 含义问题，先读取 `ads-semantic://profile/{app}?business_date={date}`。只有 `published + ready` binding 可作为正式解释；draft、missing、conflict 或 not_comparable 必须明确限制，不能根据事件名猜含义。
4. 用 `ads_data_health` 检查 ready、partial、stale、mapping_incomplete、missing 和权限。
5. 用 `ads_data_query` 读取实际映射或配置，不从名称、旧缓存或截图猜结果。
6. 平台通用概念通过 `ads_knowledge_search` 查官方知识；判断当前账户或对象时仍必须读取 config evidence，官方概念不能替代当前配置。
7. 输出候选、冲突、影响范围、缺失证据和验证方法，并区分已确认、合理推测、待验证。
8. 没有正式写入接口时只输出草案/待确认，不声称配置已生效。

## 边界

- 用户定义不能覆盖 Catalog 的正式口径、权限、审批或 evidence。
- App 不是权限资源。语义 Profile 属于组织私有 evidence；只读取当前 principal 已授权 source/provider scope 中可观察的 App facet，客户端不得自报 organization 扩大范围。
- TikTok Config/Hierarchy、Asset shared-performance 和 App identity 必须保留 health、snapshot provenance 与 limitation；Asset shared envelope 不能称为素材归因。
- TikTok `placement_type/placements`、bid strategy/amount、budget 与 budget owner（CBO/ABO）来自
  `tiktok_config_status_hierarchy`；字段为 null 时先按合同判断该策略是否本就不要求金额，不能直接宣称漏采。
- TikTok 实际版位 spend/CPI/CPS/CPM/CPC/CTR 来自 `tiktok_placement_af_cost`，不得从配置版位推导；配置历史比较使用
  `tiktok_entity_temporal_evidence`，小时级字段变化使用 `tiktok_config_change_events`。
- `tiktok_config_change_events.occurred_at_utc` 是 observation 检测时间而非平台 mutation time；只有结构和字段语义有效、字段覆盖稳定、分页完备且可比较的 observation 下，零事件才表示未检测到变化。对象未出现在后续 inventory 不等于 DELETE，只有显式返回的 inactive/delete 状态可证明该变化。
- 不能因为字段缺失补默认值；输出 cannot_judge 或明确缺失原因。
- 只读核查，不修改平台、不发送 IM、不写 registry。
