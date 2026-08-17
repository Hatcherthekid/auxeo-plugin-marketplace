---
name: ads-monitoring
description: 使用系统模板或用户规则配置并解释广告监控与告警，覆盖余额、投放状态、政策与配置变更、数据刷新和报告投递阻断；支持预览与回测，不直接发送 IM 或执行平台写操作。
---

# Ads · 监控与告警

本 Skill 回答“现在有没有必须处理的风险”和“要监控什么”。可以使用系统模板或整理用户自定义规则，但应优先复用已经通过 Ads Read 暴露的结构化 payload、registry、ReportArtifact 或 MCP evidence，不重新生成普通日报。

旧飞书监控脚本只作为已有 producer 和输出形态的定位线索；未通过 Ads Read 或 artifact 暴露的事实必须标记“当前不可读”，不能用私有路径或记忆冒充证据。

## 工作流程

1. 确认对象、渠道、业务时区、数据最新日期和用户目标。
2. 选择已登记模板，或把用户规则整理为 MonitorConfig 草案；不能从自然语言直接声称已发布。
3. 绑定正式 source、字段、grain、窗口、freshness 和质量门槛，并用 Catalog/Describe 校验。
4. 先生成 preview/backtest，展示触发、未触发、无法判断样例和预计通知量。
5. 用户确认后才进入 publish/version；没有写入接口时停在草案。
6. 运行时读取受治理 payload/registry，必要时用 `ads_data_health`、`ads_data_query` 和 `ads_workspace_analyze` 补证据。
7. 输出必须处理、需要核查、当前正常和无法判断，并保留 artifact/evidence ref。

## MonitorConfig 最小字段

`scope`、`source/metric`、`grain`、`condition`、`window/baseline`、`freshness gate`、`severity`、`cooldown`、`dedupe key`、`owner/route`、`timezone`、`enabled`、`version`。

用户阈值可以改变触发条件，但不能改变正式指标语义、权限、source authority 或 cannot_judge。模板、配置、回测、发布版本、运行状态和 delivery receipt 是不同对象。

## 输出与边界

- 每个告警包含严重级别、对象、业务日期、触发事实、freshness、影响、人工下一步和证据引用。
- 缺事实、registry 失效、日期不匹配或 threshold 缺失时输出无法判断/投递阻断，不能把缺失当作正常。
- 不直接发送飞书/IM，不静默修改规则、通知对象或调度。
- 告警是待核查信号，不自动证明经营因果；不根据名称猜预算、状态或政策。
