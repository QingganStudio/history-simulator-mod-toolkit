---
title: 超事件（super_events）
description: 超事件数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 超事件（`super_events`）

> 依赖层级：第 4 层；文件：`global/super_events.csv`；编辑器：主从详情视图；最近生成：2026-10-30。

## 用途与编辑入口

定义独立展示的超事件及其清理后条件。

- 主键：`code_name`
- 上游依赖：无显式外键。
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：无额外表级规则；仍执行字段、主键、外键、大小和 CSV 通用检查。

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 事件ID | 文本 | 必填 | — | 必填；不得缺失或使用空值；主键，表内唯一 | 超事件的唯一编号 |
| `title` | 标题 | 文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件标题 |
| `description` | 描述 | 长文本 | 必填 | — | 必填；不得缺失或使用空值 | 事件描述文本 |
| `cleaned_condition` | 结构化条件 | 结构化类型 `condition_list` | 必填 | — | 必填；不得缺失或使用空值 | 触发条件的结构化 JSON |


## 最小合法数据

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 示例记录

| 字段 | 示例值 |
| --- | --- |
| `code_name` | superevent_001 |
| `title` | 阉党失势 朝野震动 |
| `description` | 把持朝政多年的“九千岁”魏忠贤及其党羽遭到清算，阉党势力大不如前。朝野上下初时欢欣鼓舞，以为圣主临朝，中兴有望。 |
| `cleaned_condition` | JSON（1 条条件，详见下方详情） |

### 长内容详情

:::details 查看「cleaned_condition」完整值

```json
[
  {
    "column_name": "is_alive",
    "row_name": "3001_weizhongxian",
    "table_name": "ministers",
    "type": "str_judge",
    "value": "false"
  }
]
```

:::

:::details 查看原始 CSV

````csv
code_name,title,description,cleaned_condition
事件ID,标题,描述,结构化条件
超事件的唯一编号,事件标题,事件描述文本,触发条件的结构化 JSON
,,,
superevent_001,阉党失势 朝野震动,把持朝政多年的“九千岁”魏忠贤及其党羽遭到清算，阉党势力大不如前。朝野上下初时欢欣鼓舞，以为圣主临朝，中兴有望。,"[{""column_name"":""is_alive"",""row_name"":""3001_weizhongxian"",""table_name"":""ministers"",""type"":""str_judge"",""value"":""false""}]"
````

:::

## 资源与验证

超事件图片在资源编辑器中管理。

- 预览支持：已接入有限 Mock；从有效记录中随机展示一条，不判断真实条件或概率。
- 验证边界：保存时先检查必填、类型、主键、外键和专项规则。页面 Mock 只证明本地文件能被当前预览适配器读取；本地导入不会把本表写入服务端。真实数据库与玩法需等待 Workshop 服务端场景就绪。

返回：[数据表总览](./)。
