---
title: db_info
description: 逻辑 prompt db_info 的层组合、渲染形态、消费名分级与坑位提示
---

<!-- 此文件由 scripts/generate-prompt-docs.mjs 生成，请勿手工编辑。 -->
> 最近生成：2026-11-13；来源：prompt-registry.v1.json（vendorSync 锁定）。查实时元信息与原文用 mod-toolkit 的 vanilla_prompt 工具。

# db_info

## 层组合与登记

| 层 | 官方默认 | 说明 |
|---|---|---|
| setting | ✔ | 设定层（可编辑） |
| rules | — | 官方无此层（mod 可开新槽，sanguo 实证） |
| example | — | 官方无此层（mod 可开新槽，sanguo 实证） |

- 磁盘形态 `prompt/{layer}/db_info.md`；manifest 登记键 `db_info.{layer}`（flat 条目键 `db_info`）
- example 槽登记 `required: false`；setting/rules 为 `required: true`

## 渲染形态与消费分级

- 渲染形态：**static（原样使用——写 {{ }} 不会渲染，会原样发给模型）**
- 消费分级：**fallback（回退代码内硬编码默认，功能可用但质量受损）**——回退后端硬编码 _DB_INFO_PROMPT（含 ScenarioMeta 术语替换）

## 坑位提示

- 数据库表结构文档：14 表字段语义、军事落库强约束、推演世界规则；经 System 的 {db_info} 占位整体注入；字段口径铁律：外键列真实名以 _id 结尾，db_info 与一切 prompt 层文本统一写 _code_name 别名（regime_id→regime_code_name、location_province_id→location_province_code_name、party_id→party_code_name），与服务端工具契约参数同名以提高 function call 命中率；原版 17,740 code points（上限的 88.7%，距 18,000 预警线约 260 字），扩写前先看预算

## 字符预算与相关检查

- 单文件 ≤ 20,000 code points（>18,000 时 validate 侧 approaching-limit 预警）
- 剧本 mod 缺本条目层文件时：validate/preflight 报 prompt-layer；写错变量名报 prompt-render-residual

