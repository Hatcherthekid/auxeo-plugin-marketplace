---
name: ads-read-analysis
description: 自由查询和分析当前授权广告数据，支持汇总、比较、排序、诊断和动态下钻；查询计划与续查路径由服务端合同驱动。
---

# Ads · 自由分析

## 入口

每个新任务先调用 `ads_capability_context`。它返回当前 principal 可见的资源、能力、版本、health、typed guidance 和兼容状态。App 是分析 facet，不是授权资源；不得要求用户先提供固定账户 ID 或预设 App 名称。

若当前 Host 能读取 Auxeo MCP Resources、但没有披露依赖的自定义 Tool，先读取 `ads-contract://host-tool-fallback-v1`，再按其中的 `ads-query://execute/{tool_name}{?arguments}` Resource Template 调用同一只读能力。该入口仅用于 Host 恢复，不扩大权限，也不得绕过正常 Tool。

## 工作方式

遵循 `Dictionary-first -> Query-plan-first -> SQL/Python execution -> LLM interpretation`：

1. 把自然语言问题拆成范围、日期、grain、指标、维度、比较和期望输出。
2. 按服务端 guidance 发现 catalog、正式定义、可查询字段与 source authority；不在 Skill 记忆工具清单、source ID、公式或平台对象树。
3. 先执行最小充分查询；大范围问题先汇总或排序，再选择有信息增量的对象下钻。
4. 每次结果都检查 coverage、freshness、limitations、definition evidence 和 continuation。
5. 已从 capability context 选定物理资源时，数据健康调用必须携带对应 `resource_refs`，避免把单账户检查扩大为同渠道全部授权账户扫描。
6. 续查只能使用服务端返回的 opaque references、父对象 keys 与允许的 transition；不同投放结构可返回不同关系、切分或 profile。
7. `next_resource_cursor` 表示授权物理资源尚未扫描完，必须继续资源分页后才能声称“全部账户”；它与对象下钻 continuation 不同。
8. 需要复杂计算时，在 Host 允许的 Workspace SQL/Python 框架内处理已返回证据，不越权读取底层存储。
9. 先回答结论，再给时间范围、证据、口径、反证、限制和下一步。

## 计数口径

- 用户明确说事件次数、event count 或 PV 时使用 `total_events`；明确说人数、用户数或 UV 时使用 `unique_users`。
- 泛问审批率、通过率、转化率或好人率时默认使用 AppsFlyer UV，并主动说明；只问申请量、审批量等模糊数量时，先解释 event count 会重复计算同一用户的多次动作，再询问用户要次数还是人数。
- 累计多日审批率、通过率、转化率或好人率必须使用 install-cohort UV，默认 D0，用户明确指定时使用 D1/D3；actual UV 只用于单一 actual event date 或按 date 分组的逐日趋势，不得用多日 actual/activity Daily UV Sum 形成一个累计漏斗率。
- 多日或多对象 UV 相加必须称为 `Daily UV Sum / 每日 UV 之和`，并明确说明同一用户跨日可能重复计算，它不是整个日期范围内真正去重的 UV。不得简称为 UV、区间独立用户数或去重用户数。

## 边界

- 模型能看到的数据由权限裁剪决定，模型本身不决定权限。
- 正式指标由服务端计算；不自行重算或用名称猜映射。
- 未知、未采集、无权限、同步失败、不可比和真实零必须区分。
- 证据不足时明确 `cannot_judge`；不执行广告平台写入或外部发送。
