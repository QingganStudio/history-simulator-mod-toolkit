---
title: 国策树（perks_table）
description: 国策树数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 国策树（`perks_table`）

> 依赖层级：第 4 层；文件：`mongo/perks_table.csv`；编辑器：专用流程编辑器；最近生成：2026-10-30。

## 用途与编辑入口

定义国策节点、前置/互斥关系、效果和画布数据。

- 主键：`code_name`
- 上游依赖：无显式外键。
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：`perk-relations`

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 技能序号 | 文本 | 必填 | — | 必填；不得缺失或使用空值；主键，表内唯一 | 国策节点的唯一编号（Skill.#### 形态），前置、互斥引用与图标命名使用同值 |
| `category` | 技能类别 | 枚举：`管理`、`学识`、`军事`、`政治`、`随机开局` | 必填 | — | 必填；不得缺失或使用空值 | 国策分类：管理、学识、军事、政治、随机开局；主前置与互斥关系要求同分类 |
| `name` | 技能名称 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 国策的显示名称 |
| `prerequisites` | 前置技能 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 前置国策编号列表（逗号分隔）；Skill.0000 表示无前置；副前置允许跨分类 |
| `main_prerequisite` | 主前置技能 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 主前置国策；必须与当前国策同分类，且已包含在 prerequisites 中 |
| `mutually_skill` | 互斥国策 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 互斥国策编号列表（逗号分隔）；关系必须双向登记且双方同分类 |
| `description` | 技能描述 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 国策效果的说明文案 |
| `current_turn_effect` | 本回合效果 | 结构化类型 `effect_list` | 可选 | — | 可选；留空时使用空单元格 | 本回合结构化数据库效果（JSON 数组） |
| `effect_display` | 效果展示 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 效果的玩家可见展示文案 |
| `context_text` | 上下文文本 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 本回合后续推演上下文 |
| `long_term_prompt` | 长期 prompt | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 后续回合的长期语义提示词 |


## 最小合法数据

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 示例记录

| 字段 | 示例值 |
| --- | --- |
| `code_name` | Skill.1001 |
| `category` | 管理 |
| `name` | 苏复经济凋敝 |
| `prerequisites` | Skill.0000 |
| `main_prerequisite` | — |
| `mutually_skill` | — |
| `description` | 起草一组由国家主导的宏观经济干预指导纲领，旨在通过整顿财政、稳定币值与市场秩序，为后续的全面改革奠定基础。 |
| `current_turn_effect` | JSON（16 条效果，详见下方详情） |
| `effect_display` | 提高北直隶、南直隶、山西、山东、河南、陕西、浙江、江西、湖广、四川、福建、广东、广西、云南、贵州、辽东（关宁锦）税收完成率各1%。 |
| `context_text` | — |
| `long_term_prompt` | — |

### 长内容详情

:::details 查看「current_turn_effect」完整值

```json
[
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1001_beijing",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1002_nanjing",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1003_shanxi",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1004_shandong",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1005_henan",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1006_shaanxi",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1007_zhejiang",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1008_jiangxi",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1009_huguang",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1010_sichuan",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1011_fujian",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1012_guangdong",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1013_guangxi",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1014_yunnan",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1015_guizhou",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  },
  {
    "column_name": "tax_efficiency",
    "op": "add",
    "row_name": "1016_liaodongS",
    "table_name": "provinces",
    "type": "database",
    "value": 0.01
  }
]
```

:::

:::details 查看原始 CSV

````csv
code_name,category,name,prerequisites,main_prerequisite,mutually_skill,description,current_turn_effect,effect_display,context_text,long_term_prompt
技能序号,技能类别,技能名称,前置技能,主前置技能,互斥国策,技能描述,本回合效果,效果展示,上下文文本,长期 prompt
国策节点的唯一编号（Skill.#### 形态），前置、互斥引用与图标命名使用同值,国策分类：管理、学识、军事、政治、随机开局；主前置与互斥关系要求同分类,国策的显示名称,前置国策编号列表（逗号分隔）；Skill.0000 表示无前置；副前置允许跨分类,主前置国策；必须与当前国策同分类，且已包含在 prerequisites 中,互斥国策编号列表（逗号分隔）；关系必须双向登记且双方同分类,国策效果的说明文案,本回合结构化数据库效果（JSON 数组）,效果的玩家可见展示文案,本回合后续推演上下文,后续回合的长期语义提示词
,,,,,,,,,,
Skill.1001,管理,苏复经济凋敝,Skill.0000,,,起草一组由国家主导的宏观经济干预指导纲领，旨在通过整顿财政、稳定币值与市场秩序，为后续的全面改革奠定基础。,"[{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1001_beijing"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1002_nanjing"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1003_shanxi"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1004_shandong"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1005_henan"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1006_shaanxi"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1007_zhejiang"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1008_jiangxi"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1009_huguang"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1010_sichuan"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1011_fujian"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1012_guangdong"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1013_guangxi"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1014_yunnan"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1015_guizhou"",""table_name"":""provinces"",""type"":""database"",""value"":0.01},{""column_name"":""tax_efficiency"",""op"":""add"",""row_name"":""1016_liaodongS"",""table_name"":""provinces"",""type"":""database"",""value"":0.01}]",提高北直隶、南直隶、山西、山东、河南、陕西、浙江、江西、湖广、四川、福建、广东、广西、云南、贵州、辽东（关宁锦）税收完成率各1%。,,
````

:::

## 资源与验证

锁定和解锁图标在国策资源入口中管理。

- 预览支持：已接入有限交互 Mock；生成国策节点和前置边并支持基本解锁，不执行真实条件、互斥、效果或长期影响。
- 验证边界：保存时先检查必填、类型、主键、外键和专项规则。页面 Mock 只证明本地文件能被当前预览适配器读取；本地导入不会把本表写入服务端。真实数据库与玩法需等待 Workshop 服务端场景就绪。

返回：[数据表总览](./)。
