---
name: app-conversion-diagnosis
description: 判断买量效果变差更像广告流量、App 转化、AppsFlyer 归因数据、业务政策还是混合问题，并设置广告动作 Gate。用于同一 App 多渠道同时恶化、广告成本看似异常但下游漏斗变化、需要判断是否应继续优化广告，或需要解释 D0/D1 转化范围时。
---

# App 转化诊断

只解释受治理 `AppConversionDiagnosis`。方向变化和指标由 SQL/Python Analysis Product 产生，本 Skill 不重算 CPI、CPS、通过率或用户质量。

如果问题来自日报 section，先按
`docs/03_analysis_decision/PAID_GROWTH_DAILY_REVIEW_ENTRYPOINTS_V1.md` 绑定父 report 与 anchor，再补最小范围的
App Analysis Artifact；不能绕开父 report 直接把局部信号写回日报。

## 输入检查

1. 使用同一 App、business date、business timezone 和 cohort window。
2. 默认 D0；D1 必须显式，且检查 cohort maturity。
3. AppsFlyer 是 install 和经营漏斗的事实基线；媒体只补花费、点击、展示、配置和状态。
4. 保留 `af_with_media / af_only / media_only`，不能丢弃其中一种。
5. 至少需要两个可比渠道才能判断 App-wide 范围；否则输出 cannot-judge。
6. attribution、mapping 或 freshness blocked 时，进入工程修复/回补链；业务责任域保持 `cannot_judge`，不继续做广告根因判断。

## 责任域解释

- 多个可比渠道同时变弱：责任域候选为 `app_conversion`，广告动作 `restricted`。
- 只有单渠道变弱、其他渠道稳定或变强：责任域候选为 `paid_acquisition`，广告动作可以 `open`，但具体动作仍需对象证据。
- 媒体花费与 AF attribution 不守恒、mapping 或 cohort 不可比：业务责任域候选为 `cannot_judge`，广告动作 `blocked`；另生成工程 repair signal，不把数据缺口写成经营主因。
- 业务规则、审核或放款政策证据明确：责任域候选为 `business_policy`；没有证据不要猜具体政策。
- 信号冲突：责任域候选为 `mixed` 或 `cannot_judge`，保留反证。

这些判断只做责任域和范围分流，不定位具体页面、接口、风控规则或 App 版本，除非存在对应证据源。

## 输出

必须给出：

1. scope classification：`cross_channel_app_wide / single_channel / single_object / mixed / cannot_judge`。
2. responsible domain candidate。
3. ads action gate：`open / restricted / blocked`。
4. 各渠道受治理方向观察、证据和反证。
5. 不能判断项及需要补充的数据。

不直接提出预算幅度、出价、素材淘汰、衰退阈值、复制重建或 Campaign 替换规则。
