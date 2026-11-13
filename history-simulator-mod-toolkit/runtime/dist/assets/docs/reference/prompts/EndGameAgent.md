---
title: EndGameAgent
description: 逻辑 prompt EndGameAgent 的层组合、渲染形态、消费名分级与坑位提示
---

<!-- 此文件由 scripts/generate-prompt-docs.mjs 生成，请勿手工编辑。 -->
> 最近生成：2026-11-13；来源：prompt-registry.v1.json（vendorSync 锁定）。查实时元信息与原文用 mod-toolkit 的 vanilla_prompt 工具。

# EndGameAgent

## 层组合与登记

| 层 | 官方默认 | 说明 |
|---|---|---|
| setting | ✔ | 设定层（可编辑） |
| rules | ✔ | 规则层（平台统一维护） |
| example | ✔ | 示例层（可选） |

- 磁盘形态 `prompt/{layer}/EndGameAgent.md`；manifest 登记键 `EndGameAgent.{layer}`（flat 条目键 `EndGameAgent`）
- example 槽登记 `required: false`；setting/rules 为 `required: true`

## 渲染形态与消费分级

- 渲染形态：**static（原样使用——写 {{ }} 不会渲染，会原样发给模型）**
- 消费分级：**fallback（回退代码内硬编码默认，功能可用但质量受损）**——回退内联默认

## 坑位提示

- 终局评价：生平/结局/功过/庙号/未来推演 + 0-100 打分，输出严格 JSON；庙号字表（六类）在 setting 层维护——换皮剧本必改

## 字符预算与相关检查

- 单文件 ≤ 20,000 code points（>18,000 时 validate 侧 approaching-limit 预警）
- 剧本 mod 缺本条目层文件时：validate/preflight 报 prompt-layer；写错变量名报 prompt-render-residual

