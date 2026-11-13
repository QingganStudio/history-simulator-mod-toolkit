---
title: 随机事件（random_events）
description: 随机事件数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 随机事件（`random_events`）

> 依赖层级：第 4 层；文件：`mongo/random_events.csv`；编辑器：主从详情视图；最近生成：2026-10-30。

## 用途与编辑入口

定义带条件、选项和后续关系的随机事件。

- 主键：`code_name`
- 上游依赖：无显式外键。
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：`event-key-unique`

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 事件 key | 文本 | 必填 | — | 必填；不得缺失或使用空值；主键，表内唯一 | 随机事件的唯一编号 |
| `title` | 事件标题 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件标题 |
| `description` | 事件文本 | 长文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件描述文本 |
| `cleaned_condition` | 触发条件 | 结构化类型 `condition_list` | 可选 | — | 可选；留空时使用空单元格 | 触发条件的结构化 JSON |
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

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 事件概览

| 字段 | 示例值 |
| --- | --- |
| `code_name` | event.1001 |
| `title` | 海舟疑踪 |
| `description` | 长文本（98 字，详见下方详情） |
| `cleaned_condition` | JSON（1 条条件，详见下方详情） |

### 事件详情

:::details 查看「description」完整值

```text
今日收到登莱巡抚奏报，沿海卫所于昨日卯时在长山岛附近海域截获一艘小型渔船。该船船身涂满桐油，舱底暗格藏有罗盘与空白海图，船上五名"渔民"皆操闽浙口音，却无渔具与渔获。卫所已将人船暂扣，候圣裁定夺。
```

:::

:::details 查看「cleaned_condition」完整值

```json
[
  {
    "column_name": "regime_id",
    "row_name": "1004_shandong",
    "table_name": "provinces",
    "type": "str_judge",
    "value": "2001_ming"
  }
]
```

:::

### 选项 1

| 字段 | 示例值 |
| --- | --- |
| `c1_content` | "严审深挖"将渔民押解至京，交锦衣卫北镇抚司详查，船内物品封存待验。 |
| `c1_display` | 农民叛乱风险+1 |
| `cleaned_c1_effect` | JSON（1 条效果，详见下方详情） |

#### 完整值

:::details 查看「cleaned_c1_effect」完整值

```json
[
  {
    "column_name": "rebellion_risk",
    "op": "subtract",
    "row_name": "7002_peasant",
    "table_name": "social_classes",
    "type": "database",
    "value": 1
  }
]
```

:::

### 选项 2

| 字段 | 示例值 |
| --- | --- |
| `c2_content` | "宽待安抚"释放渔民，赐米粮安抚，令其返籍后不得出海。 |
| `c2_display` | 国库存银-500两；农民满意度+1 |
| `cleaned_c2_effect` | JSON（2 条效果，详见下方详情） |

#### 完整值

:::details 查看「cleaned_c2_effect」完整值

```json
[
  {
    "column_name": "treasury",
    "op": "subtract",
    "table_name": "finances",
    "type": "database",
    "value": 500
  },
  {
    "column_name": "satisfaction",
    "op": "add",
    "row_name": "7002_peasant",
    "table_name": "social_classes",
    "type": "database",
    "value": 1
  }
]
```

:::

### 选项 3

| 字段 | 示例值 |
| --- | --- |
| `c3_content` | "薄惩示警"责打渔民二十棍，渔船没收充公，通报沿海卫所加强戒备。 |
| `c3_display` | 农民满意度-1 |
| `cleaned_c3_effect` | JSON（1 条效果，详见下方详情） |

#### 完整值

:::details 查看「cleaned_c3_effect」完整值

```json
[
  {
    "column_name": "satisfaction",
    "op": "subtract",
    "row_name": "7002_peasant",
    "table_name": "social_classes",
    "type": "database",
    "value": 1
  }
]
```

:::

:::details 查看原始 CSV

````csv
code_name,title,description,cleaned_condition,c1_content,c1_display,cleaned_c1_effect,c2_content,c2_display,cleaned_c2_effect,c3_content,c3_display,cleaned_c3_effect
事件 key,事件标题,事件文本,触发条件,选项1 内容,选项1 展示,选项1 效果,选项2 内容,选项2 展示,选项2 效果,选项3 内容,选项3 展示,选项3 效果
随机事件的唯一编号,事件标题,事件描述文本,触发条件的结构化 JSON,选项1 的内容文本,选项1 的效果展示文本,选项1 的结构化效果 JSON,选项2 的内容文本,选项2 的效果展示文本,选项2 的结构化效果 JSON,选项3 的内容文本,选项3 的效果展示文本,选项3 的结构化效果 JSON
,,,,,,,,,,,,
event.1001,海舟疑踪,"今日收到登莱巡抚奏报，沿海卫所于昨日卯时在长山岛附近海域截获一艘小型渔船。该船船身涂满桐油，舱底暗格藏有罗盘与空白海图，船上五名""渔民""皆操闽浙口音，却无渔具与渔获。卫所已将人船暂扣，候圣裁定夺。","[{""column_name"":""regime_id"",""row_name"":""1004_shandong"",""table_name"":""provinces"",""type"":""str_judge"",""value"":""2001_ming""}]","""严审深挖""将渔民押解至京，交锦衣卫北镇抚司详查，船内物品封存待验。",农民叛乱风险+1,"[{""column_name"":""rebellion_risk"",""op"":""subtract"",""row_name"":""7002_peasant"",""table_name"":""social_classes"",""type"":""database"",""value"":1}]","""宽待安抚""释放渔民，赐米粮安抚，令其返籍后不得出海。",国库存银-500两；农民满意度+1,"[{""column_name"":""treasury"",""op"":""subtract"",""table_name"":""finances"",""type"":""database"",""value"":500},{""column_name"":""satisfaction"",""op"":""add"",""row_name"":""7002_peasant"",""table_name"":""social_classes"",""type"":""database"",""value"":1}]","""薄惩示警""责打渔民二十棍，渔船没收充公，通报沿海卫所加强戒备。",农民满意度-1,"[{""column_name"":""satisfaction"",""op"":""subtract"",""row_name"":""7002_peasant"",""table_name"":""social_classes"",""type"":""database"",""value"":1}]"
````

:::

## 资源与验证

随机、固定回合和链式随机事件共享对应的事件资源规则。

- 预览支持：已接入有限事件 Mock；只取有限样例展示条件和选项，不执行真实概率、后续链和效果。
- 验证边界：保存时先检查必填、类型、主键、外键和专项规则。页面 Mock 只证明本地文件能被当前预览适配器读取；本地导入不会把本表写入服务端。真实数据库与玩法需等待 Workshop 服务端场景就绪。

返回：[数据表总览](./)。
