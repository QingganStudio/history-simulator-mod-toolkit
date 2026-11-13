---
title: AgentLoop-Simulate-Step2
description: 逻辑 prompt AgentLoop-Simulate-Step2 的层组合、渲染形态、消费名分级与坑位提示
---

<!-- 此文件由 scripts/generate-prompt-docs.mjs 生成，请勿手工编辑。 -->
> 最近生成：2026-11-13；来源：prompt-registry.v1.json（vendorSync 锁定）。查实时元信息与原文用 mod-toolkit 的 vanilla_prompt 工具。

# AgentLoop-Simulate-Step2

## 层组合与登记

| 层 | 官方默认 | 说明 |
|---|---|---|
| setting | — | 官方无此层（mod 可开新槽，sanguo 实证） |
| rules | ✔ | 规则层（平台统一维护） |
| example | — | 官方无此层（mod 可开新槽，sanguo 实证） |

- 磁盘形态 `prompt/{layer}/AgentLoop-Simulate-Step2.md`；manifest 登记键 `AgentLoop-Simulate-Step2.{layer}`（flat 条目键 `AgentLoop-Simulate-Step2`）
- example 槽登记 `required: false`；setting/rules 为 `required: true`

## 渲染形态与消费分级

- 渲染形态：**static（原样使用——写 {{ }} 不会渲染，会原样发给模型）**
- 消费分级：**fatal（缺失即抛异常，无回退——裁剪内容面的硬下限）**——三阶段推演装配无 try/except，缺失直接抛

## 坑位提示

- 阶段二落库：数据锚定与业务核对，纯剧本语义；禁工具契约词（后端契约测试禁止）
- 官方无 setting 层，但 mod 可开新槽

## 字符预算与相关检查

- 单文件 ≤ 20,000 code points（>18,000 时 validate 侧 approaching-limit 预警）
- 剧本 mod 缺本条目层文件时：validate/preflight 报 prompt-layer；写错变量名报 prompt-render-residual

