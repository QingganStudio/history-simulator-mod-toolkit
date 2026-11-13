---
title: 8.1 六类事件与标识命名空间
description: 根据触发方式选择事件表，并避免随机事件标识和图片冲突
---

# 8.1 六类事件与标识命名空间

先选择正确的表，再填写条件和效果。把固定回合事件做成普通随机事件，或把连续剧情拆成互不关联的随机事件，会让后续验证变得困难。

## 如何选择

| 需求 | 使用 |
| --- | --- |
| 保存一条常规历史或推演事件 | `normal_events` |
| 满足条件后参与随机抽取 | `random_events` |
| 在指定回合触发 | `fixed_turn_events` |
| 选项会进入下一节点 | `random_events_chain` |
| 以独立大画面展示重大事件 | `super_events` |
| 让玩家主动执行并记录次数 | `decisions` |

字段明细分别见数据表参考中的六张事件表。

## 标识唯一规则

每张表首先受自身主键唯一约束。除此之外，以下三张表还有跨表规则：

```text
random_events
fixed_turn_events       ── code_name 共用一个命名空间
random_events_chain
```

例如 `event.tax.001` 已存在于 `random_events` 时，不能再用于固定回合或链式随机事件。内容预检会报告 `event-key-duplicate`。

这三张表也共用 `client-assets/dynamic-assets/events/random/` 图片目录。图片文件名把 `code_name` 中的点号换成连字符（`event.tax.001` → `event-tax-001.webp`）——点号被拉平意味着 `random.tax.001` 与 `random-tax-001` 会解析到同一张图，命名时连字符形态也应视为已占用。跨表唯一同时避免同名图片覆盖。

## 推荐命名

标识应稳定、可搜索，并能看出剧情族，例如：

```text
random.tax_shortage.001
fixed.coronation.001
chain.border_crisis.1
decision.reform_grain_tax
super.capital_falls
```

命名前缀不是运行时类型判断依据，但能减少复制记录后的冲突。复制事件后第一步应修改 `code_name`。

配图命名跟随运行时换算：常规事件直接用 `code_name`（`normal.tax_relief` → `normal.tax_relief.webp`），超事件同理；随机、固定回合和链式事件则把点号换成连字符（`random.tax_shortage.001` → `random-tax_shortage-001.webp`，`chain.border_crisis.1` → `chain-border_crisis-1.webp`）。三表图片都放进 `client-assets/dynamic-assets/events/random/`。

## 各表最小职责

### 常规事件

`normal_events` 主要保存名称、描述、效果文字和分类。它没有三个选项和结构化触发条件，不应拿来替代随机事件。

### 随机、固定回合和链式随机事件

三者拥有相近列集合：标题、描述、条件，以及三组内容、展示和效果。空选项可以不配置，但已填写的选项必须同时检查玩家文字和机器效果。

固定回合事件的 `turn` 条件只能使用 `equal`。链式随机事件的后继关系则保存在选项效果中的 `follow_up`。

### 超事件

`super_events` 保存标识、标题、描述和条件。图片不在 CSV 中，通过超事件资源目录按 `code_name` 命名。

### 决议

`decisions` 把可见条件、执行条件、重复规则、效果、提示词和可选超事件快照组合成一条玩家操作。

## 改名和删除

改事件标识时同时检查：

- 链式事件选项中的 `follow_up`；
- 决议条件中的 `decision_id` 或 `event_id`；
- 超事件快照的 `event_id`；
- 图片文件名；
- 提示词和展示文字中的人工引用。

专用事件链编辑器会清理链内的部分后继引用，但不会替你迁移其他类型事件或文案中的标识。

下一篇：[8.2 条件编辑器与判断规则](./conditions)。
