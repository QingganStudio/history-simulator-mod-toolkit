# 工具链与 Agent 工作流

这部分介绍 Mod Maker 编辑器之外的另一条官方制作路径：把校验、预检、查询工具装进你自己的 AI 编码 agent（Claude Code、Codex、ZCode 等），或直接在终端与 CI 里使用同名 CLI。

两类作者适合从这里开始：

- 想让 AI 按自己的世界观设定生成、修改 Mod 数据，并随时用官方校验兜底的作者；
- 不经编辑器、直接编辑文件，需要在命令行或持续集成里跑校验与发布预检的作者。

建议先读[工具链总览](./overview)理解这套工具的工作方式——它是 linter 模式：agent 用自己的文件编辑能力直接修改 Mod 目录，内核只做只读校验、查询与脚手架，质量来自「改动 → 校验 → 修复」的循环密度。接入方式见[Agent Plugin（插件形态）](./agent-plugin)——五种编码 agent 宿主一键安装，任意 MCP 宿主与 CLI 的用法也在那一页。想直接让 agent 上手时，读[Agent 工作流契约](./workflow)；逐个工具的输入输出查[工具参考](./tools-reference)；发布前的两道门与 severity 语义见[发布门禁与 preflight](./preflight-and-publish)。

这条路径与编辑器路径不冲突：两者产出同一套 Mod 目录契约（`manifest.yaml`、CSV 数据表、注册 JSON、提示词与 `custom.css`），最终都收敛到同一道 Workshop 上传预检。已熟练使用编辑器的作者，也可以只把 CLI 当作命令行校验器使用。
