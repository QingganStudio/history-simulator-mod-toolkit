---
title: 国策树总览
description: 使用专用画布制作节点、前置、互斥、效果和图标
---

# 国策树总览

国策树由 `perks_table` 的节点记录和节点之间的引用关系组成。编辑器画布帮助维护这些关系，但画布坐标不是 Mod 内容；游戏页面会根据服务端返回的节点和边重新布局。

## 本章专题

| 目标 | 专题 |
| --- | --- |
| 理解节点字段和分类 | [9.1 节点字段、分类与稳定标识](./perk-tree/nodes-and-categories) |
| 建立主前置和副前置 | [9.2 前置关系、连线与布局](./perk-tree/prerequisites-and-layout) |
| 维护互斥并安全删除节点 | [9.3 互斥关系与引用清理](./perk-tree/mutual-exclusion) |
| 配置实际效果、提示词和图片 | [9.4 效果、提示词与锁定/解锁图标](./perk-tree/effects-and-icons) |
| 验证游戏端解锁与提交 | [9.5 Preview 与真实解锁验证](./perk-tree/validation) |

## 一条国策包含什么

`perks_table` 每行是一条国策，包含：

- 身份：`code_name`、`category`、`name`；
- 关系：`prerequisites`、`main_prerequisite`、`mutually_skill`；
- 内容：`description`；
- 即时效果：`current_turn_effect`；
- 玩家说明：`effect_display`；
- 推演上下文：`context_text`、`long_term_prompt`；
- 图片：不在 CSV 中，按 `code_name` 派生文件名。

除身份字段外，多数字段当前可选，但“可选”只表示内容预检允许为空，不表示一条空白国策具备完整玩法价值。

## 推荐制作流程

1. 规划四个正式分类中的路线和稳定标识。
2. 新增节点，填写名称和描述。
3. 先建立主前置，再补副前置。
4. 配置互斥并检查是否与前置路径冲突。
5. 配置数据库效果、展示文字和提示词。
6. 上传锁定与解锁图标。
7. 自动布局用于整理编辑画布。
8. 保存并处理关系校验。
9. 用 Preview 检查基本节点和图标，再在 Workshop 场景验证真实解锁。

![国策树编辑器的节点画布、关系连线和详情区域](/screenshots/stage-4/19-perk-tree-editor.webp)

*图：画布用于维护节点关系和本地编辑位置，正式游戏会按关系重新布局。*

## 重要边界

- 编辑器位置保存在当前浏览器的 `localStorage`，不会写进 CSV、Manifest 或 Workshop 内容。
- 自动布局只修改编辑器位置，不修改国策关系。
- 本地 Preview 会生成基本节点和前置边，但当前不会把 `mutually_skill` 适配到游戏页面，也不会执行数据库效果。
- 本地 `local_` Mod 仍使用官方服务端场景，不能据此验收真实国策表。

从[9.1 节点字段、分类与稳定标识](./perk-tree/nodes-and-categories)开始。下一章：[地图与省份](/resources-and-ui/map-and-provinces)。
