---
title: 27 张数据表参考
description: 按依赖层级查看全部可编辑源表和自动生成的派生表
---

<!-- 此文件由 scripts/generate-table-docs.mjs 生成，请勿手工编辑。 -->

# 27 张数据表参考

这里列出 **27 张可编辑源表**。字段、顺序、类型和必填标记均来自当前共享 CSV Schema。

从第 0 层到第 4 层编辑可以减少悬空外键。每张 CSV 的前四条逻辑记录依次是英文字段名、中文显示名、字段说明和映射名，正式数据从第 5 条逻辑记录开始。

另有 [2 张派生表](./derived-tables)，只在保存时自动生成，不进入编辑器导航。

预览能力并不相同：23 张源表会被游戏页面 Mock 直接消费；`random_events_chain` 只有快照/磁盘读取而没有页面消费者；`fixed_turn_events` 与 `special_simulations` 当前未接入游戏预览。`game_state_rules` 同样未接入游戏预览。具体转换和限制见每张表的“资源与验证”。

## 第 0 层

- [政权（`regimes`）](./regimes)
- [党派（`parties`）](./parties)
- [社会阶层（`social_classes`）](./social_classes)
- [土地税率（`land_tax_configs`）](./land_tax_configs)
- [装备（`equipments`）](./equipments)
- [船只配置（`ship_configs`）](./ship_configs)
- [特殊推演（`special_simulations`）](./special_simulations)
- [初始任务（`initial_tasks`）](./initial_tasks)
- [事件预测表（`events_prediction`）](./events_prediction)
- [玩家初始属性（`player_attributes_init_table`）](./player_attributes_init_table)

## 第 1 层

- [大臣（`ministers`）](./ministers)
- [兵种配置（`arm_configs`）](./arm_configs)
- [举荐大臣（`recommend_ministers`）](./recommend_ministers)

## 第 2 层

- [省份（`provinces`）](./provinces)
- [省份土地（`province_land_holdings`）](./province_land_holdings)
- [举荐基础数据（`recommend_base`）](./recommend_base)

## 第 3 层

- [建筑（`buildings`）](./buildings)
- [军队（`troops`）](./troops)
- [财政值（`finances`）](./finances)

## 第 4 层

- [常规事件（`normal_events`）](./normal_events)
- [随机事件（`random_events`）](./random_events)
- [固定回合事件（`fixed_turn_events`）](./fixed_turn_events)
- [信息感知（`game_state_rules`）](./game_state_rules)
- [链式随机事件（`random_events_chain`）](./random_events_chain)
- [超事件（`super_events`）](./super_events)
- [决议事件（`decisions`）](./decisions)
- [国策树（`perks_table`）](./perks_table)
