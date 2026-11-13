---
title: 8.5 决议、重复策略与超事件快照
description: 配置玩家可见决议、执行规则、提示词和执行后超事件
---

# 8.5 决议、重复策略与超事件快照

决议是一条完整的玩家操作定义：什么时候看得到、什么时候能执行、执行后改什么、能否重复，以及是否展示一个超事件。

## 决议组成

| 字段组 | 作用 |
| --- | --- |
| `code_name`、`category`、`title`、`description`、`sort_order` | 身份、分组、文字和排序 |
| `visibility_conditions` | 是否出现在列表中 |
| `execution_conditions` | 当前是否可点击执行 |
| `repeat_policy` | 执行一次或长期执行 |
| `database_effects` | 实际数据修改 |
| `effect_display`、`condition_display` | 玩家看到的说明列表 |
| `short_term_prompt`、`long_term_prompt` | 后续推演语义 |
| `super_event` | 执行后展示的超事件快照 |

完整字段约束见 [`decisions`](/reference/tables/decisions)。

## 重复策略

编辑器、Toolkit 与上传预检支持四种策略，决议均由玩家主动执行，频率规则不会自动扣费或触发。

| type | 含义 | 参数 |
| --- | --- | --- |
| once | 生命周期执行一次（默认） | 无 |
| once_per_turn | 每回合最多一次 | 可选 max_total；max_per_turn 只允许省略或 1 |
| cooldown | 冷却后可再次执行 | 必填 cooldown_turns；可选 max_per_turn、max_total |
| repeatable | 条件满足时可重复执行 | 可选 max_per_turn、max_total |

新参数写正整数数字，上限留空表示不限制，不要用 0 表示无限。cooldown_turns=3，第2回合执行后，第4回合还需等待1回合，第5回合可再次执行。

可解释的旧数值会提供兼容提示，合法兼容提示不阻断保存或上传。未知策略及非法参数必须修复；修改无关字段不会擅自改写原策略。切换策略时草稿保留旧参数以便切回，保存只输出当前策略有效的参数。

## 数据效果与提示词

决议使用数据库效果修改正式数据。`effect_display` 应逐项对应真实修改，让玩家能预判结果。

短期提示词用于执行后的近期语义，长期提示词用于后续持续影响。提示词不会代替数据库效果；数据库效果也不会自动生成自然语言上下文。

## 选择超事件

决议的 `super_event` 不是只保存一个外键，而是保存快照：

```json
{
  "event_id": "super.reform_complete",
  "title": "改制完成",
  "description": "……",
  "image_url": "……"
}
```

选择超事件时，编辑器从当前 `super_events` 记录和图片信息复制这些字段。之后源超事件的标题、描述或图片改变，决议快照不会自动更新。

属性面板会提示快照与源记录不一致，并提供“同步超事件最新内容”。源记录被删除时，应重新选择或清空，不要保留失去来源的旧快照。

## 超事件本体

`super_events` 的 `code_name`、标题、描述和条件定义超事件本身。图片位于：

```text
client-assets/dynamic-assets/events/super-events/<code_name>.<扩展名>
```

条件决定真实场景是否触发；决议快照则表达某条决议执行后要展示的固定内容，两者不能互相替代。

## Preview 行为

决议 Preview 按 `sort_order` 排序，模拟四策略的总次数、本回合次数和冷却。可以推进预览回合或手动重置。同 ID 配置热更新保留历史；新会话和更换 Mod 重置。条件不会查询真实数据库，效果不会持续修改其他页面。正式扣费、条件、存档和幂等仍需真实游戏验收。

超事件 Preview 也可能从有效记录中选择一条展示，不代表其真实条件已满足。

下一篇：[8.6 Preview 与真实场景验证](./validation)。
