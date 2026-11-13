---
title: 链式随机事件（random_events_chain）
description: 链式随机事件数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 链式随机事件（`random_events_chain`）

> 依赖层级：第 4 层；文件：`mongo/random_events_chain.csv`；编辑器：专用流程编辑器；最近生成：2026-10-30。

## 用途与编辑入口

定义具有前后节点关系的随机事件链。

- 主键：`code_name`
- 上游依赖：无显式外键。
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：`event-key-unique`

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 事件 key | 文本 | 必填 | — | 必填；不得缺失或使用空值；主键，表内唯一 | 链式随机事件的唯一编号；随机事件相关表的 code_name 跨表唯一，同链节点建议共用前缀 |
| `title` | 事件标题 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件标题 |
| `description` | 事件文本 | 长文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件描述文本 |
| `cleaned_condition` | 触发条件 | 结构化类型 `condition_list` | 可选 | — | 可选；留空时使用空单元格 | 触发条件的结构化 JSON |
| `c1_content` | 选项1 内容 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 选项1 的内容文本 |
| `c1_display` | 选项1 展示 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 选项1 的效果展示文本 |
| `cleaned_c1_effect` | 选项1 效果 | 结构化类型 `effect_list` | 可选 | — | 可选；留空时使用空单元格 | 选项1 的结构化效果 JSON；可含至多一个 follow_up 指向同链后继节点 |
| `c2_content` | 选项2 内容 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 选项2 的内容文本 |
| `c2_display` | 选项2 展示 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 选项2 的效果展示文本 |
| `cleaned_c2_effect` | 选项2 效果 | 结构化类型 `effect_list` | 可选 | — | 可选；留空时使用空单元格 | 选项2 的结构化效果 JSON；可含至多一个 follow_up 指向同链后继节点 |
| `c3_content` | 选项3 内容 | 文本 | 可选 | — | 可选；留空时使用空单元格 | 选项3 的内容文本 |
| `c3_display` | 选项3 展示 | 长文本 | 可选 | — | 可选；留空时使用空单元格 | 选项3 的效果展示文本 |
| `cleaned_c3_effect` | 选项3 效果 | 结构化类型 `effect_list` | 可选 | — | 可选；留空时使用空单元格 | 选项3 的结构化效果 JSON；可含至多一个 follow_up 指向同链后继节点 |


## 最小合法数据

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 事件概览

| 字段 | 示例值 |
| --- | --- |
| `code_name` | chain_horse.1 |
| `title` | 京营马瘟 |
| `description` | 长文本（99 字，详见下方详情） |
| `cleaned_condition` | JSON（1 条条件，详见下方详情） |

### 事件详情

:::details 查看「description」完整值

```text
南京京营游击将军递上急奏：营中战马三日来突发疫病，已倒毙百二十匹，病马仍在蔓延，马厩腥臭熏天。营中现存战马仅余七成，操练与巡防皆受阻滞，恳请陛下急调应天卫所备用马匹补充，并遣太医院兽医赴营查验隔离。
```

:::

:::details 查看「cleaned_condition」完整值

```json
[
  {
    "column_name": "regime_id",
    "row_name": "1002_nanjing",
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
| `c1_content` | "内调省费，薄惩主官" 从本地驿站调用驿马充军，京营指挥使罚俸禄三个月 |
| `c1_display` | 南京京营士气-10 国库存银+1000两 |
| `cleaned_c1_effect` | JSON（2 条效果，详见下方详情） |

#### 完整值

:::details 查看「cleaned_c1_effect」完整值

```json
[
  {
    "column_name": "morale",
    "op": "subtract",
    "row_name": "4002_nanjing",
    "table_name": "troops",
    "type": "database",
    "value": 10
  },
  {
    "column_name": "treasury",
    "op": "add",
    "table_name": "finances",
    "type": "database",
    "value": 1000
  }
]
```

:::

### 选项 2

| 字段 | 示例值 |
| --- | --- |
| `c2_content` | "急调军马，严令检疫" 命令应天卫所三日内凑齐军马300匹补足京营；并遣太医院御医查探； |
| `c2_display` | 国库存银-5000两 ;下一回合触发【马厩调查】 |
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
    "value": 5000
  },
  {
    "type": "follow_up",
    "follow_up": "chain_horse.2a"
  }
]
```

:::

### 选项 3

| 字段 | 示例值 |
| --- | --- |
| `c3_content` | "严惩主官，自行处理” 责令南直隶自行处理，将京营主官贬为参将； |
| `c3_display` | 权威+1;下一回合触发【知情不报】 |
| `cleaned_c3_effect` | JSON（2 条效果，详见下方详情） |

#### 完整值

:::details 查看「cleaned_c3_effect」完整值

```json
[
  {
    "column_name": "prestige",
    "op": "add",
    "row_name": "2001_ming",
    "table_name": "regimes",
    "type": "database",
    "value": 1
  },
  {
    "type": "follow_up",
    "follow_up": "chain_horse.2b"
  }
]
```

:::

:::details 查看原始 CSV

````csv
code_name,title,description,cleaned_condition,c1_content,c1_display,cleaned_c1_effect,c2_content,c2_display,cleaned_c2_effect,c3_content,c3_display,cleaned_c3_effect
事件 key,事件标题,事件文本,触发条件,选项1 内容,选项1 展示,选项1 效果,选项2 内容,选项2 展示,选项2 效果,选项3 内容,选项3 展示,选项3 效果
链式随机事件的唯一编号；随机事件相关表的 code_name 跨表唯一，同链节点建议共用前缀,事件标题,事件描述文本,触发条件的结构化 JSON,选项1 的内容文本,选项1 的效果展示文本,选项1 的结构化效果 JSON；可含至多一个 follow_up 指向同链后继节点,选项2 的内容文本,选项2 的效果展示文本,选项2 的结构化效果 JSON；可含至多一个 follow_up 指向同链后继节点,选项3 的内容文本,选项3 的效果展示文本,选项3 的结构化效果 JSON；可含至多一个 follow_up 指向同链后继节点
,,,,,,,,,,,,
chain_horse.1,京营马瘟,南京京营游击将军递上急奏：营中战马三日来突发疫病，已倒毙百二十匹，病马仍在蔓延，马厩腥臭熏天。营中现存战马仅余七成，操练与巡防皆受阻滞，恳请陛下急调应天卫所备用马匹补充，并遣太医院兽医赴营查验隔离。,"[{""column_name"":""regime_id"",""row_name"":""1002_nanjing"",""table_name"":""provinces"",""type"":""str_judge"",""value"":""2001_ming""}]","""内调省费，薄惩主官"" 从本地驿站调用驿马充军，京营指挥使罚俸禄三个月",南京京营士气-10 国库存银+1000两,"[{""column_name"":""morale"",""op"":""subtract"",""row_name"":""4002_nanjing"",""table_name"":""troops"",""type"":""database"",""value"":10},{""column_name"":""treasury"",""op"":""add"",""table_name"":""finances"",""type"":""database"",""value"":1000}]","""急调军马，严令检疫"" 命令应天卫所三日内凑齐军马300匹补足京营；并遣太医院御医查探；",国库存银-5000两 ;下一回合触发【马厩调查】,"[{""column_name"":""treasury"",""op"":""subtract"",""table_name"":""finances"",""type"":""database"",""value"":5000},{""type"":""follow_up"",""follow_up"":""chain_horse.2a""}]","""严惩主官，自行处理” 责令南直隶自行处理，将京营主官贬为参将；",权威+1;下一回合触发【知情不报】,"[{""column_name"":""prestige"",""op"":""add"",""row_name"":""2001_ming"",""table_name"":""regimes"",""type"":""database"",""value"":1},{""type"":""follow_up"",""follow_up"":""chain_horse.2b""}]"
````

:::

## 资源与验证

链式事件图片在资源编辑器中管理。

- 预览支持：会进入编辑器快照并可从磁盘解析，但当前没有游戏页面适配器消费；预览不能推进事件链。
- 验证边界：快照/磁盘可读不代表页面已消费本表；使用专用编辑器和内容预检检查关系，真实链式推进等待 Workshop 服务端场景。

返回：[数据表总览](./)。
