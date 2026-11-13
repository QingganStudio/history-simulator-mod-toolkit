# History Simulator Mod Toolkit

《历史模拟器》的 Mod 制作工具，支持 Claude Code、Codex、Cursor、Kimi Code 和 ZCode。插件内置崇祯剧本数据与中文文档，可以查询制作资料、创建和校验 Mod，并在发布前检查内容。

## 功能

- **查资料**：离线查询内置剧本、字段定义、数据表和制作文档。
- **做 Mod**：技能覆盖从零建剧本、扩表、平衡调整、事件与国策树创作、地图和资源覆盖、提示词改写等流程。
- **检查作品**：使用校验、发布前预检和质量检查工具定位问题；提供 `/mod-validate`、`/mod-preflight`、`/mod-doctor` 命令（名称可能带宿主前缀）。
- **查游戏日志**：游戏启动并登录后，可查询当前玩家自己的日志。

## 使用要求

- 安装 **Node.js ≥ 20**，并确认运行 Agent 的环境能找到 `node`（运行 `node --version` 检查）。
- 日志查询需要在本机安装、启动并登录游戏；制作和校验 Mod 不需要启动游戏。

插件已包含运行时和依赖，安装后无需运行 `npm install` 或 `pnpm install`。

## 安装

按使用的宿主选择：[Claude Code](#claude-code) · [Codex](#codex) · [Cursor](#cursor) · [Kimi Code](#kimi-code) · [ZCode](#zcode)。需要本地文件时，可在本页下载 ZIP 或克隆仓库。

**目录选择**：仓库根目录用于注册插件市场；`history-simulator-mod-toolkit/` 子目录是插件本体。下文会标明各宿主使用哪一层目录。

### Claude Code

在系统终端执行：

```bash
claude plugin marketplace add QingganStudio/history-simulator-mod-toolkit
claude plugin install history-simulator-mod-toolkit@history-simulator
```

本地安装时，在**仓库根目录**把第一条命令替换为 `claude plugin marketplace add .`，再执行同一安装命令。安装后打开新会话，在 `/plugin` 面板确认插件已启用。

在 Claude Code 会话中运行以下命令自检：

```text
/history-simulator-mod-toolkit:mod-doctor
```

更新时，在终端执行 `claude plugin marketplace update history-simulator`，再执行 `claude plugin update history-simulator-mod-toolkit@history-simulator`，并开启新会话。

参考：[Claude Code 官方安装文档](https://code.claude.com/docs/en/discover-plugins)。

### Codex

使用支持 `plugin` 命令的 Codex CLI，在系统终端执行：

```bash
codex plugin marketplace add QingganStudio/history-simulator-mod-toolkit
codex plugin add history-simulator-mod-toolkit@history-simulator
```

本地安装时，在**仓库根目录**把第一条命令替换为 `codex plugin marketplace add .`。运行 `codex plugin list --marketplace history-simulator` 查看插件，再开启新会话使用。

桌面客户端也可在注册市场后重新打开应用，从插件目录选择 **History Simulator Mod Toolkit** 市场，安装并启用同名插件。若 CLI 没有 `plugin add` 子命令，使用桌面插件目录完成安装。

GitHub 市场更新使用 `codex plugin marketplace upgrade history-simulator` 刷新来源，然后在客户端更新插件；本地来源需要先更新本地文件。

### Cursor

个人本地安装先下载或克隆本仓库，再将**整个插件子目录**复制到下列位置；复制时包含隐藏目录 `.cursor-plugin/`：

| 系统 | 目标插件目录 |
|---|---|
| Windows | `%USERPROFILE%\.cursor\plugins\local\history-simulator-mod-toolkit\` |
| macOS / Linux | `~/.cursor/plugins/local/history-simulator-mod-toolkit/` |

完成后运行 **Developer: Reload Window** 或重新打开 Cursor，在 **Customize → Plugins** 中确认插件、技能和 MCP 服务已加载。企业环境还需允许本地插件导入。

Teams / Enterprise 的团队市场也可从 **Dashboard → Plugins & MCPs → Team Marketplaces → Add Marketplace → Import from Repo** 导入 `https://github.com/QingganStudio/history-simulator-mod-toolkit`，随后在 Customize 中安装插件。

更新本地安装时，使用最新版完整插件子目录替换安装副本并重新加载；使用团队市场时先刷新市场，再更新插件。

参考：[Cursor 官方插件文档](https://cursor.com/docs/plugins)。

### Kimi Code

先下载或克隆本仓库，在 Kimi Code 会话中安装**插件子目录**。将下列占位路径替换为本机实际的绝对路径；路径有空格时加引号：

```text
/plugins install <仓库绝对路径>/history-simulator-mod-toolkit
/reload
/plugins info history-simulator-mod-toolkit
```

在 `/plugins` 的 Installed 页确认插件已启用。Kimi 使用 `kimi.plugin.json` 内的技能和 MCP 配置，当前为用户级安装。本地安装会复制插件，更新原目录后需要重新执行安装和 `/reload`。

参考：[Kimi Code 官方插件文档](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/plugins)。

### ZCode

1. 先打开一个工作区，进入 **设置 → 插件**。
2. 点击 **创建 → 添加插件市场**，输入 `QingganStudio/history-simulator-mod-toolkit` 或完整 GitHub URL。本地安装时选择**仓库根目录**，或其中的根级 `marketplace.json`。
3. 在“个人”市场中找到 `history-simulator`，安装并启用 **历史模拟器 Mod 制作工具**。
4. 在插件详情或 MCP 列表中确认技能、命令和服务已加载。

更新时先在市场源面板刷新 `history-simulator`，再到已安装插件中检查更新。

参考：[ZCode 官方插件文档](https://zcode.z.ai/cn/docs/plugin)。

## 开始使用

安装后开启新会话，可以直接向 Agent 描述要做的事，例如：

> 请使用历史模拟器 Mod 制作工具，以内置崇祯剧本为参考，告诉我如何创建一个新的 Mod。

> 请检查 `<Mod 目录>`，运行校验和发布前预检，并列出需要修复的问题。

检查插件是否安装成功时，可以发送：

> 请使用历史模拟器 Mod 制作工具检查安装：查询 schema_list 和 codes，确认工具可调用，并报告插件版本。

预期版本为 **1.7.0**，制作工具 MCP 提供 **21 个规范工具 + 20 个别名，共 41 个工具**，插件包含 **10 份技能**。宿主可能为工具和命令加上插件命名空间前缀。

## 常见问题

| 现象 | 处理方式 |
|---|---|
| `node` 找不到或 MCP 子进程启动失败 | 确认 Node.js ≥ 20 且在 Agent 的 PATH 中，重新打开 Agent |
| 日志服务 `127.0.0.1:16270` 连接失败 | 启动游戏并登录；暂不使用日志时可在宿主中停用 `history-simulator-logs` |
| 找不到插件 | 核对使用的是仓库根目录还是插件子目录，刷新市场或重新打开会话；Cursor 检查本地导入是否允许 |
| 本地更新后仍是旧版本 | 宿主可能使用安装缓存，按对应宿主步骤重新安装或更新，随后重新加载会话 |
| 升级后仍显示旧的 MCP 服务名 | 刷新市场并更新或重新安装插件，再开启新会话；如曾手工注册旧服务，也需更新或移除旧配置 |

## License

MIT © 2026 青干工作室。完整条款见 [LICENSE](./LICENSE)，插件子目录也随包提供同一许可证。

第三方依赖 `image-size` 保留自己的版权声明与 [MIT 许可证](./history-simulator-mod-toolkit/runtime/node_modules/image-size/LICENSE)。
