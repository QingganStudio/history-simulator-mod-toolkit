# 高级配置

本篇面向已经完成[第一个 Mod](/getting-started/first-mod)的作者。这里的编辑器大多写入 JSON；它们和 CSV 数据表不同，却同样会在保存、预览和正式运行时被不同层级读取。请始终先在 Mod Maker 保存，再按[总览中的验证矩阵](./game-and-ui-config)选择 Preview、本地打包版或 Workshop 场景验证。

## 按目标选择专题

### 数据契约与场景配置

- [PG 表名映射](./game-and-ui-config/pg-table-mapping)：为固定 PG 表维护导出显示名；它不会重命名 CSV 或外键。
- [官职编辑器](./game-and-ui-config/minister-positions)：组织官职分组、职位及展示排序。
- [全局配置](./game-and-ui-config/global-config)：维护诏书开场语和主政权标识。

### 界面语义与帮助文本

- [字段元数据编辑器](./game-and-ui-config/field-metadata)：只覆盖表单字段的标签和说明，不改变 Schema。
- [页面操作指引](./game-and-ui-config/page-operation-guides)：为四个固定游戏路由维护 Markdown 帮助内容。

### 客户端玩法配置

- [省份指令编辑器](./game-and-ui-config/province-commands)：维护省份管理菜单的命令显示、作用域和 AI 文案。
- [历法配置](./game-and-ui-config/calendar-config)：定义回合如何显示为纪年和季节。
- [关键事件](./game-and-ui-config/key-events)：维护游戏内时间线展示的事件列表。
- [地图行为](./game-and-ui-config/map-behavior-config)：覆盖地图缩放上限等行为片段。
- [大臣聊天预设](./game-and-ui-config/minister-chat-presets)：配置精确匹配触发的大臣聊天附加提示词。
- [廷议快捷语](./game-and-ui-config/court-chat-quick-messages)：配置朝堂发送框的快捷按钮消息。

原有的[文本、对话与提示词](./texts-and-prompts)仍单独说明 Markdown 与 TXT 文本资产。修改任何 JSON 前，可先查看[文件结构参考](/reference/file-structure)和[保存与内容预检](/validate-and-publish/save-and-preflight)。
