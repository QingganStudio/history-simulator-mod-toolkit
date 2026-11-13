---
title: 基础剧本数据总览与制作顺序
description: 按依赖链制作政权、人物、省份、军队、财政和开局数据
---

# 基础剧本数据总览与制作顺序

基础剧本数据决定一局游戏开始时有哪些政权、人物、省份、军队和资源。它不是一张“总配置表”，而是多张通过 `code_name` 连接的 CSV。制作时应先建立被引用的基础记录，再建立人物、省份和军队，最后补充开局任务与推演数据。

## 本章专题

| 目标 | 专题 | 主要数据表 |
| --- | --- | --- |
| 理解加载顺序和稳定标识 | [7.1 数据依赖顺序与稳定标识](./scenario-data/dependencies-and-identifiers) | 全部基础表 |
| 建立政权和社会结构 | [7.2 政权、党派与社会阶层](./scenario-data/political-foundations) | `regimes`、`parties`、`social_classes` |
| 制作人物和举荐流程 | [7.3 大臣、官职与举荐](./scenario-data/ministers-and-recommendations) | `ministers`、`recommend_ministers`、`recommend_base` |
| 建立省份和经济底盘 | [7.4 省份、土地、建筑与财政](./scenario-data/provinces-and-economy) | `provinces`、`land_tax_configs`、`province_land_holdings`、`buildings`、`finances` |
| 建立军队和编制 | [7.5 装备、兵种、船只与军队](./scenario-data/military-data) | `equipments`、`arm_configs`、`ship_configs`、`troops` |
| 配置开局输入 | [7.6 开局属性、任务、预测与特殊推演](./scenario-data/opening-data) | `player_attributes_init_table`、`initial_tasks`、`events_prediction`、`special_simulations` |

字段类型、是否必填、枚举和数值范围以[数据表参考](/reference/tables/)为准。本章不复制整张 Schema，而是说明跨表组合、制作顺序和运行时消费方式。

## 推荐制作顺序

1. 先确定所有稳定 `code_name` 的命名规则。
2. 创建政权、党派、社会阶层、土地类型和装备等基础记录。
3. 创建大臣、举荐大臣和举荐基础数据。
4. 创建省份、土地持有、建筑和财政数据。
5. 创建兵种、船只和实际军队。
6. 补充玩家初始属性、任务、事件预测和特殊推演。
7. 保存并处理主键、必填、外键和派生输出问题。
8. 先用 Preview 检查页面组合，再在 Workshop 服务端场景中验证真实开局。

## 保存会产生什么

源表按 Manifest 登记到 `global/`、`mongo/` 或 `pg/`。此外，保存流程还会生成两张不在普通表格导航中编辑的派生表：

- `global/recommend.csv`：由 `recommend_base` 与同 `code_name` 的 `recommend_ministers` 合并；匹配不到举荐大臣的基础记录会被跳过。
- `global/minister_assets.csv`：两表派生行 + 立绘表额外行合并生成（详见 7.3）。

不要手工维护这两个派生文件。下一次从 Mod Maker 保存时，`recommend.csv` 会按当前源表重新覆盖；`minister_assets.csv` 的派生行按两表重算，但立绘表编辑器维护的额外行会随保存保留。

## Preview 能检查什么

Preview 会组合多张表生成大臣、政权、省份、军队、总览、任务、玩家属性和举荐等页面 Mock。例如省份页面不仅读取 `provinces`，还会读取政权、军队、建筑、土地持有、税率和全局配置。

这些组合适合发现断开的标识、空列表、数量级异常和展示问题，但税收、维护费、人口或财政汇总只是本地近似。`special_simulations` 当前没有游戏页面适配器；本地 `local_` Mod 也仍使用官方服务端场景。

## 完成标准

- 所有主键和外键通过内容预检。
- 被引用的 `code_name` 没有在制作中途被改名。
- 派生举荐和立绘登记文件成功生成。
- Preview 中目标政权、人物、省份和军队能相互关联。
- Workshop 场景中新建存档后，初始数据和多回合变化符合设计。

从[7.1 数据依赖顺序与稳定标识](./scenario-data/dependencies-and-identifiers)开始。下一章：[事件与决议](./events-and-decisions)。
