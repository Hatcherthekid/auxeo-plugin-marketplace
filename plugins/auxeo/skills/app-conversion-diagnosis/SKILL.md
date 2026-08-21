---
name: app-conversion-diagnosis
description: 判断效果变化更像流量、App 转化、归因数据、业务政策还是混合问题，并给出证据边界与广告动作 Gate。
---

# Ads · App 转化诊断

## 入口

新任务先调用 `ads_capability_context`。App 只用于查询和归因分析，不进入账户授权模型；同一授权资源可覆盖多个 App facet。

## 编排

1. 明确 App facet、业务日期、时区、cohort window、比较范围和目标指标。
2. 按服务端 guidance 获取归因、媒体、业务语义和数据健康证据，不在 Skill 固化事件、渠道或 source 路由。
3. 使用可比范围区分单一投放问题、跨范围 App 问题、数据问题、业务政策候选或混合信号。
4. 分开呈现支持证据、反证、缺失证据、责任域候选和动作 Gate。
5. 只有服务端 continuation 表明有信息增量时才继续下钻。

## 边界

- 正式指标和归因口径由服务端提供，模型不重算。
- 映射、freshness、cohort 或权限不满足时，受影响结论为 `cannot_judge`。
- 责任域分流不是具体根因证明，也不授权广告平台写操作。
