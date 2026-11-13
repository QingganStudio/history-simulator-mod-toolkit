---
title: 建筑（buildings）
description: 建筑数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 建筑（`buildings`）

> 依赖层级：第 3 层；文件：`pg/buildings.csv`；编辑器：主从详情视图；最近生成：2026-10-30。

## 用途与编辑入口

定义省份建筑和结构化建筑效果。

- 主键：`code_name`
- 上游依赖：`building_location` → `provinces.code_name`
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：无额外表级规则；仍执行字段、主键、外键、大小和 CSV 通用检查。

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 主键 | 文本 | 必填 | — | 必填；不得缺失或使用空值；主键，表内唯一；长度 ≤ 50 字符（服务端数据库列宽） | 建筑唯一编号 |
| `name` | 建筑名称 | 文本 | 必填 | — | 必填；不得缺失或使用空值；长度 ≤ 100 字符（服务端数据库列宽） | 建筑名称 |
| `building_location` | 建筑所在地 | 外键 → `provinces.code_name` | 必填 | — | 必填；不得缺失或使用空值；长度 ≤ 50 字符（服务端数据库列宽） | 建筑位于的省份（须为 provinces.code_name 中存在的值，长度 ≤50 字符） |
| `build_effects` | 建筑效果 | 字符串列表 | 必填 | — | 必填；不得缺失或使用空值 | 建筑造成的效果（可多个） |
| `effect_ops` | 结构化效果操作（canonical 统一格式） | 结构化类型 `effect_list` | 必填 | — | 必填；不得缺失或使用空值；子结构 building_effect_ops | 必填的结构化数据库效果数组（每行至少 1 条）。每一项必须使用 database 类型并包含非空 desc；add/subtract 的 value 必须为正数（方向由操作符表达）；目标表为 finances 时省略 row_name，其余表必须显式 row_name。服务端开局物化按同规则硬校验，违反会导致整个剧本无法开始。Mod Maker 和预览只保存及读取该列，不执行或转换效果。 |


## 最小合法数据

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 示例记录

| 字段 | 示例值 |
| --- | --- |
| `code_name` | building_0001 |
| `name` | 紫禁城 |
| `building_location` | 1001_beijing |
| `build_effects` | JSON（2 条效果，详见下方详情） |
| `effect_ops` | JSON（2 条效果，详见下方详情） |

### 长内容详情

:::details 查看「build_effects」完整值

```json
[
  "权威+10",
  "内宫用度+320000"
]
```

:::

:::details 查看「effect_ops」完整值

```json
[
  {
    "column_name": "prestige",
    "desc": "权威+10",
    "op": "add",
    "row_name": "2001_ming",
    "table_name": "regimes",
    "type": "database",
    "value": 10
  },
  {
    "column_name": "palace_cost",
    "desc": "内宫用度+320000",
    "op": "add",
    "table_name": "finances",
    "type": "database",
    "value": 320000
  }
]
```

:::

:::details 查看原始 CSV

````csv
code_name,name,building_location,build_effects,effect_ops
主键,建筑名称,建筑所在地,建筑效果,结构化效果操作（canonical 统一格式）
建筑唯一编号,建筑名称,建筑位于的省份,建筑造成的效果（可多个）,可选的结构化数据库效果数组。数组中的每一项必须使用 database 类型并包含非空 desc；desc 为仅建筑效果卡片支持的必填说明。Mod Maker 和预览只保存及读取该列，不执行或转换效果。
,,,,
building_0001,紫禁城,1001_beijing,"[""权威+10"",""内宫用度+320000""]","[{""column_name"":""prestige"",""desc"":""权威+10"",""op"":""add"",""row_name"":""2001_ming"",""table_name"":""regimes"",""type"":""database"",""value"":10},{""column_name"":""palace_cost"",""desc"":""内宫用度+320000"",""op"":""add"",""table_name"":""finances"",""type"":""database"",""value"":320000}]"
````

:::

## 资源与验证

本表没有专用图片字段；如需客户端资源，使用资源编辑器和已注册文件。

- 预览支持：已接入页面 Mock；作为省份详情关联内容，并参与总览数据构造。
- 验证边界：保存时先检查必填、类型、主键、外键和专项规则。页面 Mock 只证明本地文件能被当前预览适配器读取；本地导入不会把本表写入服务端。真实数据库与玩法需等待 Workshop 服务端场景就绪。

返回：[数据表总览](./)。
