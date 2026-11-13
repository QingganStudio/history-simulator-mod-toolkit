---
title: 信息感知（game_state_rules）
description: 信息感知数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 信息感知（`game_state_rules`）

> 依赖层级：第 4 层；文件：`mongo/game_state_rules.csv`；编辑器：主从详情视图；最近生成：2026-10-30。

## 用途与编辑入口

信息感知规则：每回合按游戏状态求值触发条件，命中时把输出内容注入 AI 推演提示词。

- 主键：共享 Schema 未声明单一主键；按字段与专项规则校验
- 上游依赖：无显式外键。
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：无额外表级规则；仍执行字段、主键、外键、大小和 CSV 通用检查。

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `title` | 规则标题 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 规则标题、同标题规则按难度区分 |
| `category` | 输出通道 | 枚举：`game_state`、`special_simulation` | 必填 | `"game_state"` | 必填；不得缺失或使用空值 | game_state=态势警告、special_simulation=特殊推演 |
| `mode` | 适用难度 | 文本 | 必填 | `"all"` | 必填；不得缺失或使用空值；子结构 difficulty_set | all 或 easy、normal、hard 的组合 |
| `cleaned_condition` | 触发条件 | 结构化类型 `condition_list` | 可选 | — | 可选；留空时使用空单元格 | 触发条件的结构化 JSON（扁平 AND 数组） |
| `content` | 输出内容 | 长文本 | 可选 | — | 可选；留空时使用空单元格；子结构 gsr_content | 输出内容模板、支持 {字段名} 与 {matched.列名} 占位符 |


## 最小合法数据

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 示例记录

| 字段 | 示例值 |
| --- | --- |
| `title` | 简单模式提示 |
| `category` | game_state |
| `mode` | easy |
| `cleaned_condition` | JSON（0 条条件，详见下方详情） |
| `content` | sysprompt:EasyModeContent |

### 长内容详情

:::details 查看「cleaned_condition」完整值

```json
[]
```

:::

:::details 查看原始 CSV

````csv
title,category,mode,cleaned_condition,content
规则标题,输出通道,适用难度,触发条件,输出内容
规则标题、同标题规则按难度区分,game_state=态势警告、special_simulation=特殊推演,all 或 easy、normal、hard 的组合,触发条件的结构化 JSON（扁平 AND 数组）,输出内容模板、支持 {字段名} 与 {matched.列名} 占位符
,,,,
简单模式提示,game_state,easy,[],sysprompt:EasyModeContent
````

:::

## 资源与验证

本表没有专用图片字段；如需客户端资源，使用资源编辑器和已注册文件。

- 预览支持：未进入预览快照，也没有游戏页面适配器；只使用编辑器和内容预检检查结构。
- 验证边界：本表当前没有游戏页面 Mock；保存时检查必填、类型、主键、外键和专项规则，真实行为等待 Workshop 服务端场景。

返回：[数据表总览](./)。
