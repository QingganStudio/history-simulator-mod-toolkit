# History Simulator Mod Toolkit

历史模拟器 Mod 制作工具：把 mod-toolkit 内核（MCP server + 技能）装进你的编码 agent。当前内置崇祯剧本数据，支持 Claude Code、Codex、ZCode、Kimi Code、Cursor 等宿主。

## 前置

- Node.js ≥ 20（MCP server 以 node 子进程启动，见 mcp.json）。
- 日志查询（history-simulator-logs）：需本机已安装游戏，且查询时游戏已启动并登录。

## 能力面

- **MCP server「history-simulator-mod-toolkit」（stdio）**：21 个规范工具 + 20 个 mod_ 前缀防撞名别名（共 41）——校验（validate）、发布门禁（preflight）、语义质量门（quality_gate）、schema/文档/原版数据/字段基线查询、引用影响面与改名落点、脚手架（mod_scaffold / derive_tables）、3 条 prompts（mod-validate / mod-preflight / mod-doctor）。
- **MCP server「history-simulator-logs」（streamable-http，本地 127.0.0.1:16270）**：查询当前登录玩家自己的服务端日志（tdata_log_search，只能查到自己）。由游戏客户端运行时在本机提供，游戏未启动时无法连接；登录凭证由游戏自动注入，无需配置任何 token。
- **10 份 Agent Skills**（balance-tuning、build-perk-tree、create-events、create-mod-from-scratch、custom-map、edit-mod-config、extend-tables、manage-mod-assets、repair-errors、rewrite-prompt-layers）。
- 捆绑原版剧本（chongzhen-1627）与中文制作文档，离线可用。

## 质量标准（交付四件套）

validate 0 error + 缺表 0（contractCoverage.missing 为空）+ preflight valid + quality-gate 未裁定 issue 为 0 且已裁定无 error。

## 版本

v1.7.0（与游戏 workspace 同步发布）。

## License

MIT © 2026 青干工作室，完整条款见 [LICENSE](./LICENSE)。第三方依赖 image-size 的原始许可证位于 [runtime/node_modules/image-size/LICENSE](./runtime/node_modules/image-size/LICENSE)。
