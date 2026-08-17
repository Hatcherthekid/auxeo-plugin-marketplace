---
name: ads-reporting
description: 使用系统报告模板或用户配置设计、生成和追问广告管理报告、优化师报告与专题报告，复用 ReportArtifact 和数据质量状态；不直接发送飞书/IM或执行广告平台写操作。
---

# Ads · 报表与汇报

这是私有团队 Remote MCP 发布版的报表入口。它只使用发布包实际暴露的 Ads Read 工具，不依赖本地 Dashboard 控制工具。

## 工作流程

1. 识别受众、渠道、cadence、日期、范围和重点。
2. 选择已登记 ReportDefinition，或整理 ReportConfig；不清楚的字段不能默认为全量。
3. 校验 source、metric binding、grain、窗口、freshness 和权限。
4. 先生成 synthetic preview，展示章节、样例图表、缺失数据和 cannot_judge。
5. 用户确认后才运行受治理生成或形成发布草案；公开 `ads_report_generate` 当前只支持已登记范围，不能宣传为任意渠道通用生成。
6. 报告事实来自 ReportArtifact / EvidencePack / Ads Read evidence；解释需要补证时调用 `ads-read-analysis`。
7. 保留 source、partial、claim/action ID 和 follow-up anchor；文字不能覆盖事实或不确定性。

## ReportConfig 最小字段

`audience`、`scope`、`sources`、`metrics/dimensions`、`grain`、`date/window`、`comparison`、`sections`、`charts`、`narrative style`、`timezone`、`cadence`、`route`、`enabled`、`version`。

## 交付边界

- 模板、用户配置、preview、ReportArtifact、editorial projection、发布版本和 delivery receipt 是不同对象。
- 报告生成、渲染、调度和发送是不同环节；本 Skill 不直接发送飞书/IM。
- 没有父报告 artifact 的追问不能伪造上下文。
- 不执行预算、出价、状态、素材或创编写操作。
