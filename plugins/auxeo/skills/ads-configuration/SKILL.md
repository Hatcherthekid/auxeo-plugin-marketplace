---
name: ads-configuration
description: 核对当前授权范围内的广告实体、指标、标签、组织上下文和交付配置；只读呈现正式定义、冲突与缺失证据。
---

# Ads · 配置核对

## 入口

新任务先调用 `ads_capability_context`。只使用响应中当前 principal 可见的资源、能力、目标 facet、destination 与 server guidance；App 是查询 facet，不是授权资源，也不与账户强绑定。

若当前 Host 能读取 Auxeo MCP Resources、但没有披露依赖的自定义 Tool，先读取 `ads-contract://host-tool-fallback-v1`，再按其中的 `ads-query://execute/{tool_name}{?arguments}` Resource Template 调用同一只读能力。该入口仅用于 Host 恢复，不扩大权限，也不得绕过正常 Tool。

## 编排

1. 明确用户要核对的对象、日期、业务语义和消费场景。
2. 按服务端 guidance 发现正式 catalog、health、context、knowledge 与配置证据，不在本 Skill 保存工具清单、source ID、字段枚举或路由。
3. 指标使用服务端返回的 Metric Definition Evidence；App 事件语义只使用 `published + ready` 的 Semantic Profile，并保留 definition/profile version。官方概念不能替代当前配置。
4. 将候选映射、冲突、版本、freshness、source refs、适用范围和缺口分开呈现。
5. 用户希望配置报告、监控或飞书时，先读取服务端声明的配置能力与 destination；没有相应能力或审批证据，只给草案，不声称生效。

## 边界

- 权限、指标口径、source authority 和审批由服务端确定，名称、历史配置或用户自报不能覆盖。
- 未知、未采集、无权限、同步失败和真实零必须区分。
- 不修改广告平台、组织配置、报告、监控或外部交付状态。
