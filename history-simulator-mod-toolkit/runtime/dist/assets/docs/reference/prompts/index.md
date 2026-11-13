---
title: Prompt 逐名参考总览
description: 14 个逻辑 prompt 的层组合、渲染形态、消费分级与 ScenarioMeta 键语义
---

<!-- 此文件由 scripts/generate-prompt-docs.mjs 生成，请勿手工编辑。 -->
> 最近生成：2026-11-13；来源：prompt-registry.v1.json + prompt-gate-mirror.v1.json + 原版 ScenarioMeta.yaml（键说明取其行内注释）。查实时原文用 vanilla_prompt 工具。

# Prompt 逐名参考总览

14 个逻辑 prompt 是后端运行时实际消费的名单——名单外的新逻辑名属惰性内容（登记了也不生效）。服务端不强制层槽矩阵：任意已知名可加 setting/example 层槽（[三层分工写作法](/advanced/prompt-reskin)）。

| 逻辑名 | 官方层组合 | 渲染形态 | 消费分级 |
|---|---|---|---|
| [ScenarioMeta](ScenarioMeta.md) | （flat yaml） | yaml | fallback |
| [AgentLoop-Simulate-System](AgentLoop-Simulate-System.md) | setting + rules + example | static | fatal |
| [AgentLoop-Simulate-Step1](AgentLoop-Simulate-Step1.md) | rules + example | static | fatal |
| [AgentLoop-Simulate-Step2](AgentLoop-Simulate-Step2.md) | rules | static | fatal |
| [AgentLoop-Simulate-Step3](AgentLoop-Simulate-Step3.md) | setting + rules + example | static | fatal |
| [CourtDiscussionPrompt](CourtDiscussionPrompt.md) | setting + rules | jinja | fallback |
| [NPCUnifiedPrompt](NPCUnifiedPrompt.md) | setting + rules | static | fallback |
| [TaskAgent](TaskAgent.md) | setting + rules + example | static | fallback |
| [OutputGenerationAgent](OutputGenerationAgent.md) | setting + rules + example | static | fallback |
| [EdictRefineAgent](EdictRefineAgent.md) | setting + rules | static | fallback |
| [EndGameAgent](EndGameAgent.md) | setting + rules + example | static | fallback |
| [EventManagementAgent](EventManagementAgent.md) | setting + rules | static | fatal |
| [db_info](db_info.md) | setting | static | fallback |
| [EasyModeContent](EasyModeContent.md) | rules | static | fallback |

## 渲染形态三分

- **jinja**：仅廷议模板（CourtDiscussionPrompt）——严格渲染，变量拼错运行时失败；
- **static**：其余全部——原样使用，`{{ }}` 不渲染会原样发给模型（fail-open）；
- **yaml**：仅 ScenarioMeta——整体解析、禁止分层。

## ScenarioMeta 键语义（改键即可换称呼/纪年/术语，无需碰锁定的模板）

| 键 | 类型 | 说明 |
|---|---|---|
| `protagonist_title` | string | 剧本元信息：主人公称呼 + 历法。集中一处，渲染期由 render_agent_prompt 注入为模板变量。 mod 作者改这里即可覆盖称呼与纪年，无需改任何模板里的条件分支。 内容为 YAML，由 app/utils/scenario_meta.py 用 yaml.safe_load 解析。 主人公称呼（称呼类） |
| `protagonist_role` | string | 主角短称谓：发言人 / 事件触发人标签用（mod 可改为 元首/领主 等） |
| `ruler_name` | string |  |
| `honorific` | string |  |
| `dynasty` | string |  |
| `era_name` | string | 历法（历法类） |
| `predecessor_label` | string | 第 0 回合的时间标签 |
| `predecessor_note` | string |  |
| `seasons` | string[] |  |
| `edict_term` | string | 玩家下达命令的文书名（mod 可改为 法令/命令 等） |
| `chronicle_style` | string | 事件压缩归档的文体（mod 可改为 白话/编年体 等） |
| `memorial_term` | (原版扩展键) | 玩家呈报的文书名（mod 可改为 章奏/公函 等） |
| `treasury_term` | (原版扩展键) | 国库账户称谓：契约/工具描述/财政文案共用（mod 可改为 府库/金库 等） |
| `privy_purse_term` | (原版扩展键) | 君主私库称谓：同上（mod 可改为 内库/御库 等） |
| `chronicle_term` | (原版扩展键) | 推演报告第 2 节的纪事标题（mod 可改为 朝政纪要/政事纪要 等） |
| `narrator_self_term` | (原版扩展键) | 报告叙述者自称的口吻名（mod 可改为 属下/臣属/元老 等） |
| `display_terms` | string-map | 奏折效果区块展示术语（mod 可覆盖；下列即缺省值，mod 按世界观改措辞） 例：罗马 mod 可把 营缮→营造、纪闻→要闻、核算效果→结算 等 |

未知顶层键服务端不拒绝（extra=allow，rome/sanguo 式增键合法）——但 validate 会提示 scenario-meta-unknown-key：确认后端有消费方，否则是死配置。
