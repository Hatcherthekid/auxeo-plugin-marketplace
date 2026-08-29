---
name: paid-growth-operations
description: 编排日常巡检、专题诊断、周期复盘、报告追问和动作跟进；以服务端能力与证据自由组合，不固化业务对象和分析路径。
---

# Ads · 付费增长运营

## 入口

新任务先调用 `ads_capability_context`，取得当前授权资源、可用能力、typed guidance、数据健康、版本和 continuation 语义。模型负责推理与编排，服务端负责权限、数据、指标和配置 authority。

若当前 Host 能读取 Auxeo MCP Resources、但没有披露依赖的自定义 Tool，先读取 `ads-contract://host-tool-fallback-v1`，再按其中的 `ads-query://execute/{tool_name}{?arguments}` Resource Template 调用同一只读能力。该入口仅用于 Host 恢复，不扩大权限，也不得绕过正常 Tool。

## 编排

1. 根据用户目标选择最小任务范围：问数、巡检、诊断、复盘、报告或配置核对。
2. 按服务端 guidance 组合发现、查询、Workspace 分析、知识、上下文与报告能力；不在 Skill 保存固定工具链、对象树、规则数量或报告路由。
3. 大范围任务先排序和分流，只对高影响对象继续补证；每一步根据返回 evidence 和 continuation 决定下一步。
4. 把事实、派生特征、观察、候选原因和待审批动作分层，不把数据缺口写成业务主因。
5. 输出当前结论、影响范围、证据与反证、限制、优先级、下一步和停止条件。

## 计数口径

- 漏斗转化率默认使用 AppsFlyer `unique_users`；事件活跃度或动作频次使用 `total_events`。用户只说申请量、审批量等模糊数量时，先解释并询问次数或人数。
- 累计多日漏斗率固定使用 install-cohort UV，默认 D0；显式 D1/D3 只能查询支持该窗口的 source。actual UV 仅用于单日 actual 或按日趋势，不得汇总成累计转化率。
- 聚合 daily/object UV 时必须展示 `Daily UV Sum / 每日 UV 之和`，并主动说明它会重复计算跨日用户，不是区间去重 UV。

## 边界

- 当前授权范围是唯一数据边界；App、名称或旧配置不能扩大权限。
- 不重算正式指标，不把候选原因升级为确定因果。
- 不执行广告平台写操作，不静默修改报告、监控、动作历史或外部交付。
