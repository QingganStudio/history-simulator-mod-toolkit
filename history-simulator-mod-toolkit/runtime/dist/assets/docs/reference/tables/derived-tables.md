---
title: 2 张派生表
description: recommend 与 minister_assets 的生成来源和字段
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 2 张自动生成的派生表

派生表属于 29 个 CSV logical names，但不属于 27 张可编辑源表。它们由 Mod Maker 保存快照时生成。其中立绘表（`minister_assets`）拥有专属编辑器入口：两表派生行只读展示，额外行可增删。直接手工改文件时，`recommend.csv` 的改动会被下次保存覆盖；`minister_assets.csv` 中两表之外的额外条目会被读取并在保存时保留。命令行 / agent 侧可用 mod-toolkit 的 `derive-tables` 按同一规则重建两张派生表：`minister_assets.csv`（缺省）与 `recommend.csv`（`--table recommend`）——幂等、未登记时补登记 manifest、与编辑器保存产物字节同构。

## `global/recommend.csv`

- 来源：`recommend_base` 与 `recommend_ministers`
- 生成规则：按 `code_name` 把每条 `recommend_base` 与 `recommend_ministers` 精确匹配；找不到人物的基础行会跳过，匹配人物的当前源表字段会序列化到派生行的 `option_effect`。
- 主键：`code_name`
- 编辑方式：不可作为普通源表手工维护；重新保存会覆盖。

| 字段 | 类型 | 要求 |
|---|---|---|
| `code_name` | 文本 | 必填 |
| `name` | 文本 | 必填 |
| `recommend_round` | 整数 | 必填 |
| `recommend_text` | 长文本 | 必填 |
| `option_display` | 文本 | 必填 |
| `option_effect` | 长文本 | 可选 |

## `global/minister_assets.csv`

- 来源：`ministers.code_name` 与 `recommend_ministers.code_name`
- 生成规则：两表 `code_name` 去 `数字_` 前缀派生只读行，与立绘表编辑器维护的额外行合并输出；派生行自动并入、额外行同名被派生占用时移除。图片文件本身不参与这张 CSV 的生成。
- 主键：`name`
- 编辑方式：立绘表编辑器维护额外行（两表之外的立绘条目）；派生行只读自动并入，重新保存按「两表现算 + 额外行」覆盖输出。

| 字段 | 类型 | 要求 |
|---|---|---|
| `name` | 长文本 | 必填 |
