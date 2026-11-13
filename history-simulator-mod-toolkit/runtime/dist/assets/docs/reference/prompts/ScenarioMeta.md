---
title: ScenarioMeta
description: 逻辑 prompt ScenarioMeta 的层组合、渲染形态、消费名分级与坑位提示
---

<!-- 此文件由 scripts/generate-prompt-docs.mjs 生成，请勿手工编辑。 -->
> 最近生成：2026-11-13；来源：prompt-registry.v1.json（vendorSync 锁定）。查实时元信息与原文用 mod-toolkit 的 vanilla_prompt 工具。

# ScenarioMeta

## 层组合与登记

| 层 | 官方默认 | 说明 |
|---|---|---|
| （flat） | ✔ | YAML 整体解析，禁止分层 |

- 磁盘形态 `prompt/{layer}/ScenarioMeta.yaml`；manifest 登记键 `ScenarioMeta.{layer}`（flat 条目键 `ScenarioMeta`）
- example 槽登记 `required: false`；setting/rules 为 `required: true`

## 渲染形态与消费分级

- 渲染形态：**yaml（整体解析，禁止分层）**
- 消费分级：**fallback（回退代码内硬编码默认，功能可用但质量受损）**——裸 get_sysprompt 直接抛；实际调用面走 fail-open 访问器（get_edict_term 等缺键回退默认词）

## 坑位提示

- 剧本元信息（称呼·历法·术语）集中配置处，渲染期注入为模板变量（含 display_terms.* 嵌套路径）；seasons 必须恰好 4 项；YAML 整体解析、禁止分层（层间分隔注释会破坏 yaml.safe_load）。改键即可换称呼/纪年/术语，无需碰锁定的模板

## 字符预算与相关检查

- 单文件 ≤ 20,000 code points（>18,000 时 validate 侧 approaching-limit 预警）
- 剧本 mod 缺本条目层文件时：validate/preflight 报 prompt-layer；写错变量名报 prompt-render-residual

