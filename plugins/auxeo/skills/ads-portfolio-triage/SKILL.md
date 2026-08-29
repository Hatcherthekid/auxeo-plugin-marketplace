---
name: ads-portfolio-triage
description: 对当前授权广告资产做跨范围巡检、排序和问题分流，并选择值得继续下钻的对象；不假定固定账户或层级结构。
---

# Ads · 组合巡检

## 入口

新任务先调用 `ads_capability_context`。由当前 principal 的资源范围决定模型能看到什么；App 只是可选分析 facet，一个账户可服务多个 App。

若当前 Host 能读取 Auxeo MCP Resources、但没有披露依赖的自定义 Tool，先读取 `ads-contract://host-tool-fallback-v1`，再按其中的 `ads-query://execute/{tool_name}{?arguments}` Resource Template 调用同一只读能力。该入口仅用于 Host 恢复，不扩大权限，也不得绕过正常 Tool。

## 编排

1. 从用户目标确定日期、比较窗口、业务指标和需要扫描的授权范围。
2. 按服务端 guidance 发现可用数据、健康状态和查询能力，不在 Skill 固化渠道、账户、对象树或分析 source。
3. 先做服务端允许的汇总与排序，再对少量高影响或异常对象补充证据。
4. 先收齐 `next_resource_cursor` 指向的资源页，再做全组合计；资源分页不是 Campaign 对象下钻。
5. 每次查询后读取服务端 continuation，按实际返回的关系、breakdown 或 profile 下钻；不能假设所有投放具有同一层级。
5. 输出优先级、影响、证据、反证、缺口和建议的下一次 probe，并说明停止条件。

## 边界

- 不用名称 join，不把局部结果冒充全账户扫描。
- 不重算正式指标，不把相关性写成因果。
- 不执行预算、出价、状态、素材或外部交付写操作。
