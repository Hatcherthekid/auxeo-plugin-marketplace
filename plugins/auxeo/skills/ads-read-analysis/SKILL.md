---
name: ads-read-analysis
description: 自由查询和分析当前授权广告数据，支持汇总、比较、排序、诊断和动态下钻；查询计划与续查路径由服务端合同驱动。
---

# Ads · 自由分析

## 入口

每个新任务先调用 `ads_capability_context`。它返回当前 principal 可见的资源、能力、版本、health、typed guidance 和兼容状态。App 是分析 facet，不是授权资源；不得要求用户先提供固定账户 ID 或预设 App 名称。

## 工作方式

遵循 `Dictionary-first -> Query-plan-first -> SQL/Python execution -> LLM interpretation`：

1. 把自然语言问题拆成范围、日期、grain、指标、维度、比较和期望输出。
2. 按服务端 guidance 发现 catalog、正式定义、可查询字段与 source authority；不在 Skill 记忆工具清单、source ID、公式或平台对象树。
3. 先执行最小充分查询；大范围问题先汇总或排序，再选择有信息增量的对象下钻。
4. 每次结果都检查 coverage、freshness、limitations、definition evidence 和 continuation。
5. 续查只能使用服务端返回的 opaque references、父对象 keys 与允许的 transition；不同投放结构可返回不同关系、切分或 profile。
6. `next_resource_cursor` 表示授权物理资源尚未扫描完，必须继续资源分页后才能声称“全部账户”；它与对象下钻 continuation 不同。
7. 需要复杂计算时，在 Host 允许的 Workspace SQL/Python 框架内处理已返回证据，不越权读取底层存储。
7. 先回答结论，再给时间范围、证据、口径、反证、限制和下一步。

## 边界

- 模型能看到的数据由权限裁剪决定，模型本身不决定权限。
- 正式指标由服务端计算；不自行重算或用名称猜映射。
- 未知、未采集、无权限、同步失败、不可比和真实零必须区分。
- 证据不足时明确 `cannot_judge`；不执行广告平台写入或外部发送。
