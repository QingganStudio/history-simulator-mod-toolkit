---
title: 7.2 政权、党派与社会阶层
description: 建立政权归属、党派关系和社会结构等上游基础数据
---

# 7.2 政权、党派与社会阶层

政权、党派和社会阶层是人物、省份、总览和事件效果的上游。应在录入大臣和省份前完成，并先确定主政权。

## 三张表的职责

| 表 | 用途 | 常见下游 |
| --- | --- | --- |
| `regimes` | 政权身份、名称、统治者和基础政策数据 | 省份、军队、政权页面、总览 |
| `parties` | 党派身份、名称和基础属性 | 大臣、举荐大臣、人物页面 |
| `social_classes` | 社会阶层及其影响、满意度等初始数据 | 总览、事件和国策效果 |

字段明细见 [`regimes`](/reference/tables/regimes)、[`parties`](/reference/tables/parties) 和 [`social_classes`](/reference/tables/social_classes)。

## 建立政权

1. 为每个政权分配稳定且唯一的 `code_name`。
2. 填写名称、统治者和 Schema 要求的基础数值。
3. 在[13.3 全局配置](/advanced/game-and-ui-config/global-config)中把 `main_regime_code` 选择为该政权（下拉列表来自政权表，天然匹配 `code_name`）。
4. 再创建引用该政权的省份和军队。

`current_ruler` 对有明确君主的政权应填写真实人物；同一真实统治者出现在多个政权时，`quality-gate` 的 `duplicate-ruler` 会报 error。没有单一统治者的政权可以使用规范占位值，例如“各自为政”或“纷争内斗”；“纷争内斗”仅精确值豁免，不代表真实人物可以重复。

Preview 的政权和总览 Mock 会把 `regimes` 与 `provinces`、`global_config` 组合。只有政权记录而没有归属省份时，页面可以出现政权名称，但领地和汇总会为空。

## 建立党派

党派记录先于大臣记录存在。`ministers.party_id` 和 `recommend_ministers.party_id` 都引用 `parties.code_name`，不要填写党派中文名。

如果删除党派，应先把普通大臣和举荐大臣迁移到其他合法党派。只修改党派显示名不需要改动引用端。

## 建立社会阶层

社会阶层通常独立存在，不通过显示名与其他表连接，但事件、决议和国策效果可能通过表名、行名或字段名修改它们。制作完成后应保留稳定的行标识和合理的初始数值范围。

对阶层名称或标识进行重构时，不能只检查显式外键，还要搜索结构化效果中的目标表和目标行。

## 与其他基础配置的关系

土地税率和装备也属于上游字典，但分别在省份和军队专题解释：

- 土地类型与税率：[7.4 省份、土地、建筑与财政](./provinces-and-economy)
- 装备需求与兵种：[7.5 装备、兵种、船只与军队](./military-data)

## 验证清单

- `code_name` 唯一且没有临时编号。
- 主政权配置指向存在的 `regimes` 记录。
- 大臣和举荐大臣的党派下拉框能找到目标记录。
- Preview 的政权和总览页面没有出现空名称或错误归属。
- Workshop 新局中主政权、省份归属和社会阶层初值正确。

下一篇：[7.3 大臣、官职与举荐](./ministers-and-recommendations)。
