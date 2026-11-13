---
title: 7.6 开局属性、任务、预测与特殊推演
description: 配置玩家初始能力、开局任务、未来事件提示和特殊推演
---

# 7.6 开局属性、任务、预测与特殊推演

这四张表不构成政权—省份—军队的实体链，但会直接影响新局玩家状态、任务界面和推演输入。应在主体剧本稳定后配置。

## 玩家初始属性

[`player_attributes_init_table`](/reference/tables/player_attributes_init_table) 使用名称和值表达初始能力。Preview 当前识别以下中文名称并映射到页面属性：

| 表内名称 | Preview 属性 |
| --- | --- |
| 军事 | `military` |
| 管理 | `management` |
| 政治 | `politics` |
| 学识 | `knowledge` |
| 谋略（别名） | `politics` |

`谋略` 是 `politics` 的已识别别名（原版数据只使用上表四个名称）。名称拼写不在映射中时，内容预检可能仍能通过，但 Preview 不会把它写入对应玩家属性。最终还要验证服务端接受的正式名称集合。

## 初始任务

[`initial_tasks`](/reference/tables/initial_tasks) 提供开局任务及解决方案摘要。任务文字应明确描述玩家目标，不要把内部字段名或开发说明直接展示给玩家。

Preview 会把这张表适配到任务页面，可检查标题、内容和顺序；任务完成条件和真实奖励仍取决于服务端流程。

## 事件预测

[`events_prediction`](/reference/tables/events_prediction) 用于未来事件和回合提示。预测回合应与当前历法及开局回合一致，避免出现开局前的无意义预测或远超测试范围的事件。

Preview 会把它用于未来事件展示，但不会模拟真实时间推进后是否必然发生。历法显示规则另见[13.7 历法配置](/advanced/game-and-ui-config/calendar-config)。

## 特殊推演

[`special_simulations`](/reference/tables/special_simulations) 保存特殊推演条目和提示内容，字段以当前数据表参考为准。

这张表当前没有游戏页面 Preview 适配器。应通过普通主从表格、内容预检和 Workshop 场景中的实际触发流程验证，不能用“预览里没看到”判断它未保存。

## 新局验证顺序

1. 保存并确认四张表的必填、类型和数量限制通过。
2. 在 Preview 检查玩家属性、任务和未来事件展示。
3. 检查开局回合与历法配置是否一致。
4. 发布到测试用 Workshop 场景并新建存档。
5. 检查初始属性、首批任务、预测事件和特殊推演实际输入。
6. 至少推进一个完整回合，确认这些初值没有被错误重复应用。

不要使用已有存档验证初始数据；存档通常已经持久化了创建时的初值。

返回[第 7 章总览](../scenario-data)。下一章：[事件与决议](../events-and-decisions)。
