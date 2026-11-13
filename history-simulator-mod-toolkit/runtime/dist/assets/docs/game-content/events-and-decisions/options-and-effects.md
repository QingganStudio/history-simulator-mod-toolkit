---
title: 8.3 选项、效果与后续事件
description: 为事件选项配置数据库修改、玩家属性和链式后继
---

# 8.3 选项、效果与后续事件

随机、固定回合和链式随机事件最多有三组选项。每组选项由三部分组成：玩家看到的选项内容、结果展示，以及系统实际执行的效果数组。

## 三组选项字段

| 选项 | 内容 | 展示 | 效果 |
| --- | --- | --- | --- |
| 1 | `c1_content` | `c1_display` | `cleaned_c1_effect` |
| 2 | `c2_content` | `c2_display` | `cleaned_c2_effect` |
| 3 | `c3_content` | `c3_display` | `cleaned_c3_effect` |

内容是按钮或选择文字，展示是对玩家的结果说明，效果才是机器执行的数据。只填写其中一部分会产生“看起来有结果但数据未改变”或“数据改变但玩家不知道”的问题。

## 效果类型

### 数据库效果 `database`

选择目标表、行、字段和运算。例如：

```json
{
  "type": "database",
  "table_name": "finances",
  "column_name": "treasury",
  "op": "subtract",
  "value": 10000
}
```

可用运算包括 `add`、`subtract`、`multiply`、`divide`、`set`。字符串目标通常只能使用 `set`；布尔类型目标（如 `ministers.is_alive`、`provinces.is_coastal`）同样只能使用 `set`（覆盖），值为 `true` 或 `false`；除法的值不能为 `0`。

`finances` 是单行表，因此该目标必须完全省略 `row_name`。`row_name: ""`、`row_name: null`、`row_name: "Value"` 和任意其他值都不是合法输出。最新版 Mod Maker 会在打开旧文件时自动移除这些旧占位字段；直接上传未经重新保存的旧目录会被内容预检拒绝。

#### 建筑表 `effect_ops` 的附加硬规则

建筑表（`buildings.effect_ops`）在上述通用规则之外，还受以下服务端物化契约约束（保存、上传预检与开局物化三层一致硬校验，违反任意一条都会导致整个剧本无法开始）：

1. **每行至少 1 条效果**：`effect_ops` 是必填列，空数组或空单元格不允许。
2. **`add`/`subtract` 的 `value` 必须为正数**：增减方向由操作符表达，禁止用负数或 `0` 表达方向（「权威 -10」应写作 `op: "subtract", value: 10`）。
3. **目标表为 `finances` 时省略 `row_name`；其余所有表（含 `provinces`）必须显式 `row_name`**。
4. 每一项仍必须是带非空 `desc` 的 `database` 效果。

`building_location` 必须是 `provinces.code_name` 中存在的省份，且长度不超过 50 字符。

### 玩家属性效果 `attribute`

普通事件选项可以修改 `military`、`management`、`politics` 或 `knowledge` 等玩家属性。决议效果编辑器不提供这一类型，决议应使用其正式支持的数据效果。

### 后续事件 `follow_up`

只在 `random_events_chain` 中使用：

```json
{ "type": "follow_up", "follow_up": "chain.border_crisis.2a" }
```

它不是独立连线文件，而是某个选项效果数组中的一项。事件链画布的边由这些效果反向推导。

## 目标选择原则

事件和决议的结构化条件读取、数据库效果写入，只能目标下列 13 张表：

`arm_configs`、`buildings`、`equipments`、`finances`、`land_tax_configs`、`ministers`、`parties`、`provinces`、`regimes`、`ship_configs`、`social_classes`、`troops`、`province_land_holdings`。

`table_name`、`table_name_1` 和 `table_name_2` 必须写上述英文规范表名。中文名、简写、PG 别名或 `_original_table_name*` 元数据都不能代替规范表名或获得访问权。禁止表会使保存和上传硬失败；旧 Mod 不会自动迁移或豁免，已导入的错误值会保留在编辑器中供作者手动修复。

1. 先选择表；多行表再选择实际行，`finances` 不选择行且不写 `row_name`。
2. 再选择字段或 JSON 子字段。
3. 根据字段类型选择合法运算。
4. 检查值的单位和数量级。
5. 用展示文字准确描述实际变化。

表单下拉框能防止大部分拼写错误，但目标记录后续被删除时，旧效果仍可能失效。修改基础剧本标识后应重新打开相关事件检查。

## 结构化值

部分字段是数组或对象。编辑器可能显示子字段路径，保存时通过 `path` 指向内部成员。不要把整对象覆盖当作普通数值加减；如果目标结构不支持所选操作，改用专用子字段编辑方式。

## 常见错误

- 把中文显示名写成 `row_name`，真实主键无法命中。
- 对字符串或布尔目标使用加减乘除。
- 用 `divide: 0`。
- 选项文字存在，但效果数组为空。
- 删除后继节点后仍保留手工写入的 `follow_up`。
- 展示文字与真实运算方向相反。

Preview 会展示部分选项和效果文字，但不会把数据库效果持续应用到其他页面。最终应在真实场景中记录执行前后值进行对照。

下一篇：[8.4 链式随机事件编辑器](./random-event-chains)。
