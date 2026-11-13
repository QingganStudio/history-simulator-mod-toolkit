---
title: CourtDiscussionPrompt
description: 逻辑 prompt CourtDiscussionPrompt 的层组合、渲染形态、消费名分级与坑位提示
---

<!-- 此文件由 scripts/generate-prompt-docs.mjs 生成，请勿手工编辑。 -->
> 最近生成：2026-11-13；来源：prompt-registry.v1.json（vendorSync 锁定）。查实时元信息与原文用 mod-toolkit 的 vanilla_prompt 工具。

# CourtDiscussionPrompt

## 层组合与登记

| 层 | 官方默认 | 说明 |
|---|---|---|
| setting | ✔ | 设定层（可编辑） |
| rules | ✔ | 规则层（平台统一维护） |
| example | — | 官方无此层（mod 可开新槽，sanguo 实证） |

- 磁盘形态 `prompt/{layer}/CourtDiscussionPrompt.md`；manifest 登记键 `CourtDiscussionPrompt.{layer}`（flat 条目键 `CourtDiscussionPrompt`）
- example 槽登记 `required: false`；setting/rules 为 `required: true`

## 渲染形态与消费分级

- 渲染形态：**jinja（模板渲染，严格：变量拼错即渲染失败）**
- 消费分级：**fallback（回退代码内硬编码默认，功能可用但质量受损）**——渲染异常（StrictUndefined）被调用点 broad except 兜住

## 坑位提示

- 全库唯一经 Jinja2 渲染的条目（StrictUndefined）：11 个业务变量 + ScenarioMeta 全部字段（含 display_terms.*）+ time_label/time_note 可用；变量拼错运行时渲染失败；| default() 会静默兜住拼写错误

## 变量表（调用点必填）

| 变量 | 说明 |
|---|---|
| `{{ task_content }}` | 当前廷议话题 |
| `{{ minister_a_name }}` | 大臣 A 的姓名 |
| `{{ minister_a_position }}` | 大臣 A 的官职 |
| `{{ minister_a_stance }}` | 大臣 A 的政治立场 |
| `{{ minister_a_party }}` | 大臣 A 的所属党派 |
| `{{ minister_a_system_prompt }}` | 大臣 A 的角色设定文本 |
| `{{ minister_b_name }}` | 大臣 B 的姓名 |
| `{{ minister_b_position }}` | 大臣 B 的官职 |
| `{{ minister_b_stance }}` | 大臣 B 的政治立场 |
| `{{ minister_b_party }}` | 大臣 B 的所属党派 |
| `{{ minister_b_system_prompt }}` | 大臣 B 的角色设定文本 |

## 字符预算与相关检查

- 单文件 ≤ 20,000 code points（>18,000 时 validate 侧 approaching-limit 预警）
- 剧本 mod 缺本条目层文件时：validate/preflight 报 prompt-layer；写错变量名报 prompt-render-residual

