---
title: 事件与决议总览
description: 选择事件类型并正确配置条件、选项、效果、事件链和决议
---

# 事件与决议总览

事件系统包含六张用途不同的表。它们共享部分条件和效果编辑器，但触发方式、选项结构、图片目录和 Preview 支持范围并不相同。

## 本章专题

| 目标 | 专题 |
| --- | --- |
| 选择正确事件类型并规划标识 | [8.1 六类事件与标识命名空间](./events-and-decisions/event-types-and-keys) |
| 配置回合、数值、字符串和玩法状态条件 | [8.2 条件编辑器与判断规则](./events-and-decisions/conditions) |
| 配置三个选项、数据库效果和属性效果 | [8.3 选项、效果与后续事件](./events-and-decisions/options-and-effects) |
| 用专用画布维护事件链 | [8.4 链式随机事件编辑器](./events-and-decisions/random-event-chains) |
| 制作决议和执行后超事件 | [8.5 决议、重复策略与超事件快照](./events-and-decisions/decisions-and-super-events) |
| 区分编辑器、Preview 和真实服务端 | [8.6 Preview 与真实场景验证](./events-and-decisions/validation) |

## 六张表

| 表 | 主要用途 | 是否有选项 | 图片命名空间 |
| --- | --- | --- | --- |
| `normal_events` | 常规事件记录 | 否 | `events/normal` |
| `random_events` | 按条件出现的随机事件 | 最多三个 | `events/random` |
| `fixed_turn_events` | 指定回合出现的事件 | 最多三个 | `events/random` |
| `random_events_chain` | 通过选项连接的事件链节点 | 最多三个 | `events/random` |
| `super_events` | 独立大画面展示 | 否 | `events/super-events` |
| `decisions` | 玩家主动查看和执行的决议 | 执行按钮 | 可快照引用超事件图片 |

`random_events`、`fixed_turn_events`、`random_events_chain` 共用事件和图片命名空间，三张表的非空 `code_name` 必须跨表唯一。

## 推荐制作流程

1. 确定事件类型和唯一 `code_name`。
2. 填写标题、描述、分类等基础字段。
3. 配置机器可执行的条件。
4. 配置选项和效果。
5. 单独填写面向玩家的条件、效果或选项展示文字。
6. 需要串联时使用事件链或决议超事件入口。
7. 配置图片并保存。
8. 先检查结构和有限 Preview，再在 Workshop 场景验证真实触发。

## 三种内容不要混淆

- **条件结构**决定系统是否允许事件出现或决议执行。
- **效果结构**决定执行后修改哪些数据。
- **展示文字**只负责向玩家解释，不会替代条件或效果。

因此“文字上写了需要国库十万”并不会自动建立国库条件；“效果展示写了减少银两”也不会修改财政表。

固定回合事件的图片统一通过资源系统管理；CSV 只使用[当前数据表参考](/reference/tables/fixed_turn_events)列出的字段。

从[8.1 六类事件与标识命名空间](./events-and-decisions/event-types-and-keys)开始。下一章：[国策树](./perk-tree)。
