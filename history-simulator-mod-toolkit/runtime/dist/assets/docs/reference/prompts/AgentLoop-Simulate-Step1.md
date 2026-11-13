---
title: AgentLoop-Simulate-Step1
description: 逻辑 prompt AgentLoop-Simulate-Step1 的层组合、渲染形态、消费名分级与坑位提示
---

<!-- 此文件由 scripts/generate-prompt-docs.mjs 生成，请勿手工编辑。 -->
> 最近生成：2026-11-13；来源：prompt-registry.v1.json（vendorSync 锁定）。查实时元信息与原文用 mod-toolkit 的 vanilla_prompt 工具。

# AgentLoop-Simulate-Step1

## 层组合与登记

| 层 | 官方默认 | 说明 |
|---|---|---|
| setting | — | 官方无此层（mod 可开新槽，sanguo 实证） |
| rules | ✔ | 规则层（平台统一维护） |
| example | ✔ | 示例层（可选） |

- 磁盘形态 `prompt/{layer}/AgentLoop-Simulate-Step1.md`；manifest 登记键 `AgentLoop-Simulate-Step1.{layer}`（flat 条目键 `AgentLoop-Simulate-Step1`）
- example 槽登记 `required: false`；setting/rules 为 `required: true`

## 渲染形态与消费分级

- 渲染形态：**static（原样使用——写 {{ }} 不会渲染，会原样发给模型）**
- 消费分级：**fatal（缺失即抛异常，无回退——裁剪内容面的硬下限）**——三阶段推演装配无 try/except，缺失直接抛

## 坑位提示

- 阶段一推演草案：数据锚定 + 五层推演，不调工具；可开新 setting 层槽（sanguo 实证：在 setting 层修正规则层推演法则的剧本语义）；禁 Harness 契约词与旧胁迫式补丁表述（后端契约测试锁定）

## 字符预算与相关检查

- 单文件 ≤ 20,000 code points（>18,000 时 validate 侧 approaching-limit 预警）
- 剧本 mod 缺本条目层文件时：validate/preflight 报 prompt-layer；写错变量名报 prompt-render-residual

