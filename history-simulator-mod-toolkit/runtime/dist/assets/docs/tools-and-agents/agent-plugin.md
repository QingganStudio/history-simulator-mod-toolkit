---
title: 历史模拟器 Mod 制作工具（Agent Plugin）
description: 把工具链安装成 Claude Code / Codex / ZCode / Kimi Code / Cursor 插件——MCP 工具、技能与斜杠命令一体分发
---

# 历史模拟器 Mod 制作工具（Agent Plugin）

> 适用范围：mod-toolkit 的 Agent Plugin 分发线（`--plugin-mode`、Claude Code / Codex / ZCode / Kimi Code / Cursor 五宿主形态、全形态分发包与发布候选包）；最近代码核对：2026-10-22。
>
> 各宿主的插件机制随版本演进，**安装语法以宿主官方文档为最终准绳**；本页命令以写作时点的官方文档为准。

## 这是什么

对外仓库和插件 ID 均为 `history-simulator-mod-toolkit`；支持展示名的宿主显示 **History Simulator Mod Toolkit**（中文为 **历史模拟器 Mod 制作工具**）。Claude Code 的公开 marketplace ID 为 `history-simulator`，安装标识为 `history-simulator-mod-toolkit@history-simulator`。两个 MCP 服务名分别为 `history-simulator-mod-toolkit`（Mod 制作）和 `history-simulator-logs`（日志查询）。内部 `mod-toolkit` 包名保持原样；当前插件内置崇祯剧本作为制作参考。

独立插件仓库共用一份 `history-simulator-mod-toolkit/` 主体，通过五种宿主的清单和 MCP 配置接入。运行 `pnpm build:agent-plugin:claude` 即生成这份共用主体及全部薄配置；市场 ID 统一为 `history-simulator`。下表列出全形态分发包中可独立安装的宿主目录。

工具链的标准接入形态是 **agent 宿主的插件**：一次安装同时获得 MCP 工具面、任务技能与斜杠命令，不需要手工配置 MCP server。当前提供 **Claude Code、Codex、ZCode、Kimi Code、Cursor 五种宿主形态**，都从同一份可移植插件包（Agent Plugins 1.0.0 规范形态）适配生成——其中 Codex 与 Cursor 原生兼容该规范，前者零适配直装、后者只需改写一个路径变量。不便装插件时，也可以手动注册 MCP server（见下节）或直接使用命令行（CLI，退出码可当 CI 门禁）。

## 安装矩阵（五宿主 + 任意 MCP 宿主）

| 宿主 | 形态目录 | 安装 |
|---|---|---|
| **Claude Code** | `claude-code/` | `claude plugin marketplace add <claude-code 目录>` 后 `claude plugin install history-simulator-mod-toolkit@history-simulator`；或免安装试运行 `claude --plugin-dir <claude-code/history-simulator-mod-toolkit>` |
| **OpenAI Codex** | `codex/` | `codex plugin marketplace add <codex 目录>` 后经插件目录安装/启用（可移植包即 Codex 原生格式；Codex 亦兼容读取 Claude 市场清单） |
| **ZCode（智谱）** | `zcode/` | 把 `zcode/` 目录加入 ZCode Plugin Marketplace 后安装 |
| **Kimi Code（月之暗面）** | `kimi/` | Kimi 会话内 `/plugins install <kimi/history-simulator-mod-toolkit 目录>`（或推 GitHub 后以仓库 URL 安装；仅用户级安装，属宿主限制） |
| **Cursor** | `cursor/` | 复制 `cursor/history-simulator-mod-toolkit` 到 `~/.cursor/plugins/local/` 后 Reload Window；或团队市场 Import from Repo |

**任意 MCP 宿主（手动配置）**：把可移植包 `history-simulator-mod-toolkit/` 当插件根，`mcp.json` 中 `${PLUGIN_ROOT}` 替换为该目录绝对路径后抄进宿主配置：

```json
{ "mcpServers": { "history-simulator-mod-toolkit": {
    "type": "stdio",
    "command": "node",
    "args": ["<解压路径>/runtime/dist/mcp.js", "--plugin-mode"]
},
    "history-simulator-logs": {
    "type": "http",
    "url": "http://127.0.0.1:16270/"
} } }
```

安装后在 agent 里跑斜杠命令 `/mod-doctor`（或 MCP prompt `mod-doctor`）自检——应报告 schema 与错误码目录双绿。

## 日志查询（history-simulator-logs）

除 mod 制作工具外，插件还带一个 **`history-simulator-logs` MCP server**：工具 `tdata_log_search`，查询**当前登录玩家自己**的服务端日志（排障用：先 `search="ERROR"` 粗查，再用 `&&` 限定词与 trace_id 收窄；服务端自动按身份隔离，看不到他人日志）。

- **前置条件：游戏已启动且已登录**——该服务由游戏客户端运行时在本机 `127.0.0.1:16270` 提供，游戏未运行时无法连接（连接失败请先开游戏）；未登录时调用会收到明确的 401 提示。
- **无需任何 token 配置**：登录凭证由游戏自动注入，agent 与插件全程不接触 token。

### 全形态分发包

发布候选包含一个 **全形态分发包**（`history-simulator-mod-toolkit-all-forms-<版本>.zip`，配套 `.sha256` 指纹）：解压即得安装指南 README（置顶）+ 可移植包 + 五宿主形态目录，**各形态完整自包含——拖走任一目录即可独立安装**。发给任何平台的同事都开箱即用（归档条目名为规范正斜杠，macOS/Linux 标准 unzip 可正常解压）。完整性校验：

```bash
sha256sum -c history-simulator-mod-toolkit-all-forms-<版本>.zip.sha256
```

> 分发注记：全形态包与单核包均为**候选包**——文档与原版剧本内容的公开分发授权仍在审核（内容权利方），对外传播前请与发行方确认。

## 插件模式与手动注册的差异

MCP server 以 `--plugin-mode` 启动时的注册面是 **21 个规范工具名 + 20 个 `mod_` 前缀别名（共 41 个）**，并隐藏会写盘覆盖项目 `AGENTS.md` / `CLAUDE.md` 的 `guide_init`——插件形态的工作流契约由宿主技能面承载，不再往 Mod 目录落契约文件。非插件模式注册 **22 个规范名 + 21 个别名（共 43 个，含 `guide_init`）**。

两种模式都注册 3 条 MCP prompts（斜杠命令）：`/mod-validate`、`/mod-preflight`、`/mod-doctor`，做一次快速体检的入口；Claude Code 与 ZCode 形态另把它们落盘为宿主原生命令目录（Codex 原生消费 MCP prompts；Cursor/Kimi 首版不转译命令）。

## 技能面

插件携带 8 份原生任务技能（与 guide 共享：从零创建、扩表、事件与决议创作、国策树创作、地图制作、修错、平衡调整、prompt 层改写），另有**两份插件专属技能**：`edit-mod-config`（修改注册 JSON 配置）与 `manage-mod-assets`（资源覆盖全流程，含请求键约定与回退命中核对）。插件技能由宿主技能面自动发现，不经 `guide_init` 分发；技能 frontmatter 只用跨宿主核心字段，五宿主行为一致。

## 质量保障

每个宿主形态在发布前都要过专属验收：清单结构、按宿主方式展开配置并启动真实 MCP server、工具/资源/prompts 计数核对、抽样调用与捆绑原版剧本的自检——保证五种形态装到哪里行为都一致。发布候选包另过官方 schema 与内容审计、归档条目名规范断言（保证任何系统都能正常解压）与「解压后任一形态可独立安装使用」的抽取验证。插件还内置技能触发的评测目录与只读评测集，用于持续验证技能在真实 agent 里被正确触发。

## 选择哪种形态

| 形态 | 适合谁 |
|---|---|
| Agent Plugin | Claude Code / Codex / ZCode / Kimi Code / Cursor 用户——安装即得工具 + 技能 + 斜杠命令 |
| 手动 MCP 注册 | 其他支持 MCP 的 agent（配置片段见上文安装矩阵） |
| CLI | 不配置 MCP 的任何工作流（退出码可当 CI 门禁） |

工具逐项的输入输出见[工具参考](./tools-reference)；发布门禁语义见[发布门禁与 preflight](./preflight-and-publish)。
