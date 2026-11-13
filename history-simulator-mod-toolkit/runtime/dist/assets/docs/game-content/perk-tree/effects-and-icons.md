---
title: 9.4 效果、提示词与锁定/解锁图标
description: 配置国策即时效果、玩家说明、推演上下文和两态图片
---

# 9.4 效果、提示词与锁定/解锁图标

一条完整国策不仅要能解锁，还应明确修改什么数据、向玩家展示什么，并给后续推演提供准确上下文。

## 四类内容

| 字段 | 作用 | 是否直接修改数据 |
| --- | --- | --- |
| `current_turn_effect` | 本回合结构化数据库效果 | 是 |
| `effect_display` | 玩家看到的效果说明 | 否 |
| `context_text` | 本回合后续推演上下文 | 否 |
| `long_term_prompt` | 后续回合长期语义 | 否 |

不要用展示文字替代结构化效果，也不要在提示词中要求模型再次计算已经由数据库效果应用的数值，否则可能产生重复影响。

## 即时效果

国策使用与事件相似的结构化数据库效果：选择表、行、字段、运算和值。应遵循：

`current_turn_effect` 只能写入这 13 张表：`arm_configs`、`buildings`、`equipments`、`finances`、`land_tax_configs`、`ministers`、`parties`、`provinces`、`regimes`、`ship_configs`、`social_classes`、`troops`、`province_land_holdings`。`table_name` 必须是这些英文规范表名；中文名、简写、PG 别名和 `_original_table_name*` 都不能授权。禁止表会使保存和上传硬失败，旧 Mod 不会自动迁移或豁免；已导入的错误值会保留，直到作者修正。

1. 目标行在当前剧本中真实存在。
2. 运算适合字段类型：字符串和布尔类型目标只能使用 `set`（覆盖），布尔值为 `true` 或 `false`，详见[选项、效果与后续事件](../events-and-decisions/options-and-effects)。
3. 数值单位和方向与展示文字一致。
4. 对数组或对象使用受支持的子字段目标。
5. 在真实场景记录执行前后值。

## 推演文本

`context_text` 应说明本回合发生了什么，以及哪些数值已经处理；`long_term_prompt` 应描述未来持续影响。两者都应避免伪造数据库事实。

例如即时效果已经扣除国库时，上下文应明确“数值影响已经应用”，而不是要求后续推演再次扣除。

## 两态图标

每个国策可上传锁定和解锁两张图片，保存到：

```text
client-assets/dynamic-assets/skill-icons/Skill.<数字>-lock.<扩展名>
client-assets/dynamic-assets/skill-icons/Skill.<数字>-unlock.<扩展名>
```

如果 `code_name` 是 `Skill.0123`，编辑器内部资源 ID 使用 `0123`，最终文件仍是 `Skill.0123-lock` 和 `Skill.0123-unlock`。

属性面板的“画布显示：锁定/解锁”只切换编辑器预览，不修改国策状态。删除 Mod 覆盖图片后，预览会回退原版同路径资源；没有原版资源时显示为空。

## 图片更换

结构化图片入口允许更换扩展名。保存时会删除同一文件主体的旧扩展名，避免 `.png` 和 `.webp` 同时残留。重新打开工程时，注册图片扫描识别 png、jpg、jpeg 和 webp。

格式、大小及打包版限制见[第 11 章：图片和资源](/resources-and-ui/images-and-assets)。

## 一致性检查

- 锁定和解锁图标能明显区分状态。
- 两张图的尺寸和比例适合相同节点容器。
- 效果展示与实际运算逐项对应。
- 上下文不重复应用即时数值。
- ID 改名后两张图片同步改名。

下一篇：[9.5 Preview 与真实解锁验证](./validation)。
