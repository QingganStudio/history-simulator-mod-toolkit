---
title: 省份土地（province_land_holdings）
description: 省份土地数据表的当前字段、依赖、限制和验证参考
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 省份土地（`province_land_holdings`）

> 依赖层级：第 2 层；文件：`pg/province_land_holdings.csv`；编辑器：表格视图；最近生成：2026-10-30。

## 用途与编辑入口

把省份与土地类型关联并记录面积。

- 主键：`code_name`
- 上游依赖：`province_code` → `provinces.code_name`；`land_type_code` → `land_tax_configs.code_name`
- 下游引用：共享 Schema 中没有其他源表显式引用。
- 表级专项规则：无额外表级规则；仍执行字段、主键、外键、大小和 CSV 通用检查。

## 字段

字段顺序、类型、必填和默认值直接来自共享 CSV Schema；中文名与说明只合并当前 Mod Maker 字段元数据。

| 字段 | 中文名 | 类型 / 关联 | 要求 | 默认值 | 约束与空值 | 说明 |
|---|---|---|---|---|---|---|
| `code_name` | 省份土地键值 | 文本 | 必填 | — | 必填；不得缺失或使用空值；只读，由编辑器维护；主键，表内唯一 | 复合派生主键，由 province_code 与 land_type_code 以下划线拼接而成 |
| `province_code` | 省份键值 | 外键 → `provinces.code_name` | 必填 | — | 必填；不得缺失或使用空值 | 所属省份的 code_name，引用 provinces 表 |
| `land_type_code` | 土地类型键值 | 外键 → `land_tax_configs.code_name` | 必填 | — | 必填；不得缺失或使用空值 | 土地类型的 code_name，引用 land_tax_configs 表 |
| `area_mu` | 面积（亩） | 整数 | 必填 | — | 必填；不得缺失或使用空值；最小值 0；最大值 1900000000 | 该省该土地类型的登记面积 |


## 最小合法数据

以下示例取自当前通过契约校验的原版首条数据。字段中文名、说明和类型见上方“字段”；完整值可在本页详情或原始 CSV 中查看。含外键的记录必须与上游表一起存在。

## 示例记录

| 字段 | 示例值 |
| --- | --- |
| `code_name` | 1001_beijing_8001_privateland |
| `province_code` | 1001_beijing |
| `land_type_code` | 8001_privateland |
| `area_mu` | 66804583 |

:::details 查看原始 CSV

````csv
code_name,province_code,land_type_code,area_mu
code_name,province_code,land_type_code,area_mu
,,,
,,,
1001_beijing_8001_privateland,1001_beijing,8001_privateland,66804583
````

:::

## 资源与验证

本表没有专用图片字段；如需客户端资源，使用资源编辑器和已注册文件。

- 预览支持：已接入页面 Mock；参与省份土地明细、产出和税收近似计算。
- 验证边界：保存时先检查必填、类型、主键、外键和专项规则。页面 Mock 只证明本地文件能被当前预览适配器读取；本地导入不会把本表写入服务端。真实数据库与玩法需等待 Workshop 服务端场景就绪。

返回：[数据表总览](./)。
