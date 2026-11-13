---
title: 决议事件（decisions）
description: 决议事件数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 决议事件（`decisions`）

> 依赖层级：第 4 层；文件：`mongo/decisions.csv`；编辑器：主从详情视图；最近生成：2026-10-30。

## 用途与编辑入口

定义玩家可见、可执行并可重复判定的决议。

- 主键：`code_name`
- 上游依赖：无显式外键。
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：`decision-structure`

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 决议 ID | 文本 | 必填 | — | 必填；不得缺失或使用空值；主键，表内唯一 | 决议唯一 ID，建议使用 decision.xxx 命名 |
| `category` | 分类 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 决议分类标识，例如内政、军事、外交 |
| `title` | 标题 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 玩家在决议列表中看到的标题 |
| `description` | 决议描述 | 长文本 | 必填 | — | 必填；不得缺失或使用空值 | 决议用途与可执行时机的说明 |
| `sort_order` | 排序值 | 整数 | 必填 | `0` | 必填；不得缺失或使用空值 | 分类内的排序值，数值越小越靠前 |
| `visibility_conditions` | 可见条件 | 结构化类型 `json_value` | 必填 | — | 必填；不得缺失或使用空值；子结构 decision_conditions | 决定是否展示决议的 AND 条件列表；空值表示始终可见 |
| `execution_conditions` | 执行条件 | 结构化类型 `json_value` | 必填 | — | 必填；不得缺失或使用空值；子结构 decision_conditions | 决定是否允许执行的 AND 条件列表 |
| `repeat_policy` | 重复执行规则 | 结构化类型 `json_value` | 必填 | `{"type":"once"}` | 必填；不得缺失或使用空值；子结构 decision_repeat_policy | once 执行一次；once_per_turn 每回合一次；cooldown 冷却后重复；repeatable 可重复。cooldown_turns、max_per_turn、max_total 为正整数，上限留空表示不限制。 |
| `database_effects` | 数据库效果 | 结构化类型 `effect_list` | 必填 | `[]` | 必填；不得缺失或使用空值 | 点击成功后立即严格执行的 database 效果数组，仅使用 path/op/value 干净模型 |
| `effect_display` | 效果展示 | 字符串列表 | 必填 | `[]` | 必填；不得缺失或使用空值 | 面向玩家展示的效果文本数组 |
| `condition_display` | 条件展示 | 字符串列表 | 必填 | `[]` | 必填；不得缺失或使用空值 | 面向玩家展示的条件文本数组 |
| `short_term_prompt` | 短期提示词 | 长文本 | 必填 | — | 必填；不得缺失或使用空值 | 执行决议后写入短期上下文的提示词 |
| `long_term_prompt` | 长期提示词 | 长文本 | 必填 | — | 必填；不得缺失或使用空值 | 执行决议后写入长期上下文的提示词 |
| `super_event` | 执行后超事件 | 结构化类型 `json_value` | 可选 | — | 可选；留空时使用空单元格；子结构 decision_super_event | 执行成功后展示的超事件快照对象，包含 event_id、title、description、image_url |

## 重复策略子字段契约

| type | 执行频率 | 有效参数 |
|---|---|---|
| once | 执行一次 | 无 |
| once_per_turn | 每回合一次 | max_total |
| cooldown | 冷却后重复 | cooldown_turns、max_per_turn、max_total |
| repeatable | 长期执行 | max_per_turn、max_total |

参数为正整数数字，可选上限留空表示不限制；cooldown 必填 cooldown_turns，once_per_turn 的 max_per_turn 只能省略或1。

第2回合执行、冷却3回合：第4回合剩1，第5回合可再次执行

可解释旧值提供非阻断兼容提示，无关编辑保留原参数。切换策略的草稿可恢复，保存当前有效参数。

## 最小合法数据

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 示例记录

| 字段 | 示例值 |
| --- | --- |
| `code_name` | decision_001 |
| `category` | 内政 |
| `title` | 赈济陕西 |
| `description` | 命户部拨银赈济陕西，并遣官核查灾情。 |
| `sort_order` | 100 |
| `visibility_conditions` | JSON（1 个字段，详见下方详情） |
| `execution_conditions` | JSON（1 个字段，详见下方详情） |
| `repeat_policy` | JSON（1 个字段，详见下方详情） |
| `database_effects` | JSON（2 条效果，详见下方详情） |
| `effect_display` | JSON（2 条效果，详见下方详情） |
| `condition_display` | JSON（2 条条件，详见下方详情） |
| `short_term_prompt` | 皇帝本回合执行了【赈济陕西】。赈灾款已经拨付，推演必须考虑地方官绅、灾民和朝廷财政对此事的即时反应。 |
| `long_term_prompt` | 朝廷已经建立陕西赈济机制。后续涉及陕西灾情、流民和叛乱时，应考虑赈灾政策产生的持续影响。 |
| `super_event` | — |

### 长内容详情

:::details 查看「visibility_conditions」完整值

```json
{
  "all": [
    {
      "type": "turn",
      "judge_type": "more_equal",
      "value": 2
    }
  ]
}
```

:::

:::details 查看「execution_conditions」完整值

```json
{
  "all": [
    {
      "type": "num_judge",
      "table_name": "finances",
      "column_name": "treasury",
      "judge_type": "more_equal",
      "value": 200000
    },
    {
      "type": "skill",
      "skill_id": "Skill.1001"
    }
  ]
}
```

:::

:::details 查看「repeat_policy」完整值

```json
{
  "type": "once"
}
```

:::

:::details 查看「database_effects」完整值

```json
[
  {
    "type": "database",
    "table_name": "finances",
    "column_name": "treasury",
    "op": "subtract",
    "value": 200000
  },
  {
    "type": "database",
    "table_name": "provinces",
    "row_name": "1006_shaanxi",
    "column_name": "public_sentiment",
    "op": "add",
    "value": 5
  }
]
```

:::

:::details 查看「effect_display」完整值

```json
[
  "国库存银减少200000两",
  "陕西民心增加5点"
]
```

:::

:::details 查看「condition_display」完整值

```json
[
  "当前回合≥2",
  "解锁技能Skill.1001"
]
```

:::

:::details 查看原始 CSV

````csv
code_name,category,title,description,sort_order,visibility_conditions,execution_conditions,repeat_policy,database_effects,effect_display,condition_display,short_term_prompt,long_term_prompt,super_event
决议 ID,分类,标题,详细描述,排序值,可见条件,执行条件,重复执行规则,数据库效果,效果展示,条件展示,短期提示词,长期提示词,执行后超事件
决议唯一 ID,决议分类标识,玩家看到的标题,决议的详细说明,分类内排序值,决定是否展示的 AND 条件列表,决定是否允许执行的 AND 条件列表,执行一次或长期执行,点击成功后立即应用的 database 效果数组,面向玩家展示的效果文本数组,面向玩家展示的条件文本数组,下一次推演注入的短期效果,后续推演持续注入的长期效果,执行成功后展示的超事件快照对象
,,,,,,,,,,,,,
decision_001,内政,赈济陕西,命户部拨银赈济陕西，并遣官核查灾情。,100,"{""all"":[{""type"":""turn"",""judge_type"":""more_equal"",""value"":2}]}","{""all"":[{""type"":""num_judge"",""table_name"":""finances"",""column_name"":""treasury"",""judge_type"":""more_equal"",""value"":200000},{""type"":""skill"",""skill_id"":""Skill.1001""}]}","{""type"":""once""}","[{""type"":""database"",""table_name"":""finances"",""column_name"":""treasury"",""op"":""subtract"",""value"":200000},{""type"":""database"",""table_name"":""provinces"",""row_name"":""1006_shaanxi"",""column_name"":""public_sentiment"",""op"":""add"",""value"":5}]","[""国库存银减少200000两"",""陕西民心增加5点""]","[""当前回合≥2"",""解锁技能Skill.1001""]",皇帝本回合执行了【赈济陕西】。赈灾款已经拨付，推演必须考虑地方官绅、灾民和朝廷财政对此事的即时反应。,朝廷已经建立陕西赈济机制。后续涉及陕西灾情、流民和叛乱时，应考虑赈灾政策产生的持续影响。,
````

:::

## 资源与验证

本表没有专用图片字段；如需客户端资源，使用资源编辑器和已注册文件。

- 预览支持：已接入有限交互 Mock；展示、排序与四种重复策略/次数/冷却/幂等的有限模拟；可推进回合、保留同ID热改历史和手动重置；条件不求值，效果不持续写回其他页面。
- 验证边界：保存时先检查必填、类型、主键、外键和专项规则。页面 Mock 只证明本地文件能被当前预览适配器读取；本地导入不会把本表写入服务端。真实数据库与玩法需等待 Workshop 服务端场景就绪。

返回：[数据表总览](./)。
