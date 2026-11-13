---
title: 固定回合事件（fixed_turn_events）
description: 固定回合事件数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 固定回合事件（`fixed_turn_events`）

> 依赖层级：第 4 层；文件：`mongo/fixed_turn_events.csv`；编辑器：主从详情视图；最近生成：2026-10-30。

## 用途与编辑入口

定义在固定回合参与判定的事件。

- 主键：`code_name`
- 上游依赖：无显式外键。
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：`fixed-turn-event`、`event-key-unique`

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 事件 key | 文本 | 必填 | — | 必填；不得缺失或使用空值；主键，表内唯一 | 固定回合事件的唯一编号 |
| `title` | 事件标题 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件标题 |
| `description` | 事件文本 | 长文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件描述文本 |
| `cleaned_condition` | 触发条件 | 结构化类型 `condition_list` | 可选 | — | 可选；留空时使用空单元格 | 固定回合触发条件的结构化 JSON |
| `c1_content` | 选项1 内容 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 选项1 的内容文本 |
| `c1_display` | 选项1 展示 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 选项1 的效果展示文本 |
| `cleaned_c1_effect` | 选项1 效果 | 结构化类型 `effect_list` | 可选 | — | 可选；留空时使用空单元格 | 选项1 的结构化效果 JSON |
| `c2_content` | 选项2 内容 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 选项2 的内容文本 |
| `c2_display` | 选项2 展示 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 选项2 的效果展示文本 |
| `cleaned_c2_effect` | 选项2 效果 | 结构化类型 `effect_list` | 可选 | — | 可选；留空时使用空单元格 | 选项2 的结构化效果 JSON |
| `c3_content` | 选项3 内容 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 选项3 的内容文本 |
| `c3_display` | 选项3 展示 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 选项3 的效果展示文本 |
| `cleaned_c3_effect` | 选项3 效果 | 结构化类型 `effect_list` | 可选 | — | 可选；留空时使用空单元格 | 选项3 的结构化效果 JSON |


## 最小合法数据

原版该表当前没有数据行，以下示例是生成器合成的最小结构示例（`fixed.demo`），不代表任何原版事件。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或下方 CSV 中查看。含外键的记录必须与上游表一起存在。

## 事件概览

| 字段 | 示例值 |
| --- | --- |
| `code_name` | fixed.demo |
| `title` | 固定回合示例 |
| `description` | 在第 1 回合参与判定的最小示例。 |
| `cleaned_condition` | JSON（1 条条件，详见下方详情） |

### 事件详情

:::details 查看「cleaned_condition」完整值

```json
[
  {
    "type": "turn",
    "judge_type": "equal",
    "value": 1
  }
]
```

:::

### 选项 1

| 字段 | 示例值 |
| --- | --- |
| `c1_content` | — |
| `c1_display` | — |
| `cleaned_c1_effect` | — |

### 选项 2

| 字段 | 示例值 |
| --- | --- |
| `c2_content` | — |
| `c2_display` | — |
| `cleaned_c2_effect` | — |

### 选项 3

| 字段 | 示例值 |
| --- | --- |
| `c3_content` | — |
| `c3_display` | — |
| `cleaned_c3_effect` | — |

:::details 查看 CSV（4 条 metadata + 合成示例行；原版无数据行）

````csv
code_name,title,description,cleaned_condition,c1_content,c1_display,cleaned_c1_effect,c2_content,c2_display,cleaned_c2_effect,c3_content,c3_display,cleaned_c3_effect
事件 key,事件标题,事件文本,触发条件,选项1 内容,选项1 展示,选项1 效果,选项2 内容,选项2 展示,选项2 效果,选项3 内容,选项3 展示,选项3 效果
固定回合事件的唯一编号,事件标题,事件描述文本,固定回合触发条件的结构化 JSON,选项1 的内容文本,选项1 的效果展示文本,选项1 的结构化效果 JSON,选项2 的内容文本,选项2 的效果展示文本,选项2 的结构化效果 JSON,选项3 的内容文本,选项3 的效果展示文本,选项3 的结构化效果 JSON
,,,,,,,,,,,,
fixed.demo,固定回合示例,在第 1 回合参与判定的最小示例。,"[{""type"":""turn"",""judge_type"":""equal"",""value"":1}]",,,,,,,,,
````

:::

## 资源与验证

图片在资源编辑器中管理，CSV 只使用字段表列出的当前字段。

- 预览支持：未进入预览快照，也没有游戏页面适配器；使用普通主从详情视图和内容预检检查结构。
- 验证边界：本表当前没有游戏页面 Mock；保存时检查必填、类型、主键、外键和专项规则，真实行为等待 Workshop 服务端场景。

返回：[数据表总览](./)。
