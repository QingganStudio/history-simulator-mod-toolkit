---
title: 高级配置总览与验证
description: 选择十一种高级配置编辑器，并正确区分保存、预览、本地和 Workshop 验证结果
---

# 高级配置总览与验证

高级配置不是一个统一的“游戏规则开关”。十一个编辑器分别写入 Manifest JSON、客户端资源 JSON 或仅供导出使用的映射；它们的保存检查、Preview 热更新和正式消费者也不同。本页帮助你选择正确专题和验证环境；字段定义、示例和回退细节以链接的专题页为准。

## 配置地图

| 目标 | 专题 | 输出位置 | 是否进入 Manifest | 主要消费者 |
| --- | --- | --- | --- | --- |
| PG 导出显示名 | [13.1 PG 表名映射](./game-and-ui-config/pg-table-mapping) | `pg-table-mapping.json` | 是 | 导出/数据库表显示映射 |
| 官职分组与排序 | [13.2 官职编辑器](./game-and-ui-config/minister-positions) | `global/minister_positions.json` | 是 | Preview 的大臣职位 Mock |
| 诏书标题与主政权 | [13.3 全局配置](./game-and-ui-config/global-config) | 权威：`global_config.json`；客户端副本：`client-assets/global_config.json` | 仅根目录副本 | 客户端诏书页、Preview 场景 Mock |
| 字段标签和说明 | [13.4 字段元数据编辑器](./game-and-ui-config/field-metadata) | `client-assets/field-definitions.json` | 否 | SchemaStore、编辑表单 |
| 页面帮助 Markdown | [13.5 页面操作指引](./game-and-ui-config/page-operation-guides) | `client-assets/page-operation-guides.json` | 否 | 游戏内全局说明按钮 |
| 省份管理命令 | [13.6 省份指令编辑器](./game-and-ui-config/province-commands) | `client-assets/map-data/province-schema.json` | 否 | 省份管理菜单 |
| 回合显示历法 | [13.7 历法配置](./game-and-ui-config/calendar-config) | 权威：`client-assets/game-config.json`；服务端副本：`global/game_config.json` | 仅 global 副本 | 顶部回合/季节显示、月份卡片、季节 BGM 与转场视频 |
| 时间线事件 | [13.8 关键事件](./game-and-ui-config/key-events) | `client-assets/key-events.json` | 否 | 关键事件时间线 |
| 地图缩放上限 | [13.9 地图行为](./game-and-ui-config/map-behavior-config) | `client-assets/map-data/map-behavior-config.json` | 否 | 地图缩放（`maxScale`，稀疏覆盖官方基底） |
| 大臣聊天暗语提示 | [13.10 大臣聊天预设](./game-and-ui-config/minister-chat-presets) | `client-assets/minister-chat-prompt-presets.json` | 否 | 大臣单人聊天的附加提示词 |
| 朝堂快捷按钮 | [13.11 廷议快捷语](./game-and-ui-config/court-chat-quick-messages) | `client-assets/court-chat-quick-messages.json` | 否 | 朝堂群聊发送框快捷按钮 |

`client-assets/` 文件通过客户端资源覆盖被读取，不在 `manifest.yaml` 登记；四个根目录/`global/` JSON 由共享 Manifest 契约登记。历法与全局配置虽然各保存两份，但 Mod Maker 导入只读表中标记的权威文件，不会从副本回退或合并。不要手动交换路径或把客户端副本加入 Manifest。

## 统一保存与预检边界

在 Mod Maker 内完成修改后，先执行保存，再看内容预检结果。保存会把当前编辑状态写成输出快照，但它不等同于完整业务验收：**官职与全局配置的结构校验由上传预检承担**（`json-schema` 码，见 13.2 / 13.3），其中全局配置的 `main_regime_code` 还须匹配 `regimes.code_name`——该项失配在编辑器保存与上传预检都会被拦下（`json-fk-target-missing`）。但官职的非空与唯一性不做阻断。因此，预检通过能证明结构合法，不能证明官职名称唯一或职位字段非空。

PG 表名映射仍是编辑器内阻断的例外：不合法别名、非字符串中文名和全局重复别名在编辑器中就会被拦。字段元数据、页面操作指引、省份指令、历法、关键事件、地图行为、聊天预设与快捷语各有运行时解析与回退规则，且多数已由上传预检承担结构校验，应按对应专题的检查清单核对。

关闭、切换页面或重开工程前请主动保存。顶栏撤销/重做只覆盖表格数据，不能把它当作这些工具 JSON 的完整版本管理或全局离开保护。

## 选择验证环境

| 环境 | 可以确认什么 | 不能据此确认什么 |
| --- | --- | --- |
| 编辑器即时提示/全量预览 | 表单完整性、局部文案和历法连续显示 | 服务端真实数据或命令执行 |
| Mod Preview | 已接入快照的页面 Mock、部分客户端 JSON 热刷新 | 真实外部请求、数据库与完整回合规则 |
| 本地打包版 | 客户端资源覆盖、协议、重启后的资源回退 | 本地 `local_` Mod 的服务端场景；它仍使用官方场景 |
| Workshop 服务端场景 | 服务端实际场景、业务规则、AI 和多回合状态 | 不能替代本地文件/资源路径的基础检查 |

Preview 不连接真实服务端；它只读取有限的本地快照并屏蔽真实外部请求。对于本地 `local_` Mod，业务场景 ID 为 `null`，游戏仍使用官方服务端场景。因此“本地打包版页面看起来正常”并不能证明省份命令、主政权或官职已经被服务端接受。

## 推荐收尾顺序

1. 在专题编辑器内完成修改，并处理它给出的即时错误。
2. 保存 Mod，阅读[保存与内容预检](/validate-and-publish/save-and-preflight)中的结果边界。
3. 对支持 Preview 的专题检查 Mock 页面；对省份指令则保存后重载目标页。
4. 在本地打包版检查客户端资源和协议回退。
5. 只有依赖真实场景、服务端数据或命令执行的结论，才在 Workshop 场景就绪后验收。

接下来可从[13.1 PG 表名映射](./game-and-ui-config/pg-table-mapping)开始，或按上表跳到具体配置。
