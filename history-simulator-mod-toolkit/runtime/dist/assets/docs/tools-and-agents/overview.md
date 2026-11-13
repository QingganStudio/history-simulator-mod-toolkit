---
title: 工具链总览
description: mod-toolkit 的双绑定形态、linter 模式、与 Mod Maker 编辑器工作流的关系和质量模型
---

# 工具链总览

> 适用范围：mod-toolkit 内核注册面（22 个 MCP 工具 / 23 个 CLI 命令）与 `mod-toolkit/guide` 工作流契约；最近代码核对：2026-09-25；验证层级：preflight 输出与游戏上传门禁逐码一致；validate 覆盖 CSV 数据面与跨文件引用完整性（不拦上传）。

## mod-toolkit 是什么

mod-toolkit 是 Mod 制作工具内核：schema 感知的 CSV 读取、全量结构校验、发布前整目录预检、文档与原版数据查询、语义质量门、引用影响面查询、单表骨架与场景基线。它不随游戏安装，而是一个独立的命令行 / MCP 工具，供外部 agent 和终端使用。

它有两种完全等价的形态，构成**双绑定**：

- **MCP server**（stdio）：注册进 Claude Code、Codex、ZCode 等支持 MCP 的 agent，工具以原生调用方式使用；
- **CLI**：同名命令行工具（连字符为规范形态、下划线为等价别名，参数大体对等），任何 agent、终端脚本或 CI 都能直接调用。

两种形态背后是同一个内核、同一套工具面（22 个 MCP 工具对应 22 个同名 CLI 命令，CLI 另有 `vanilla-list` 目录级命令），除帮助与版本信息外全部输出结构化 JSON。带 MCP 时优先用工具调用；没有 MCP 环境时 CLI 是完全等价的兜底。

## linter 模式：内核不代写，只校验

这套工具的工作方式是 **linter 模式**：agent（或你自己）用文件编辑能力直接修改 Mod 目录里的 CSV / JSON / 提示词文件，内核对目录做只读校验与查询，并在每轮改动后立即反馈结构化问题清单。内核不替你生成整棵 Mod 数据——生成路径完全开放（任何 agent、任何编辑器都可以），校验路径收口在唯一权威。

内核会写入的只有骨架、派生表与辅助文件：`mod_scaffold` 一次生成整 Mod 毛坯（manifest、目录树、每表 CSV 的元数据块与骨架默认行，files 经保注释最小编辑同步登记——手写注释存活，不覆盖已有 CSV/骨架）、`derive_tables` 按固定规则重建派生表 minister_assets（幂等写盘并经保注释最小编辑补登记 manifest，不触碰其他文件）、两者均支持 `--dry-run` 预览变更计划零落盘、`scaffold` 输出单表骨架行（JSON 结果，由 agent 抄进 CSV）、`guide_init` 把工作流契约与 schema 快照写入目录、`baseline_set` 写入场景基线。骨架与派生表之外的编辑循环仍由你或你的 agent 直接改文件——内核不代写内容。

为什么这样设计：**质量来自循环密度，不来自一次性生成**。让模型一次生成全部数据再上线，错误会层层堆积；让每轮小改动都立刻经过 `validate` → 按 reasonCode 修复 → 再校验的循环，问题在产生当轮就被清掉。内核的角色是把这个循环变成零成本的 reflex。

## 与 Mod Maker 编辑器工作流的关系

编辑器路径（本指南第二篇到第六篇）提供表单与专用编辑器、结构化的图片槽位与字段元数据保护，保存时自动生成 `manifest.yaml`、全部注册文件和 2 张派生表，并执行保存预检。工具链路径把这些换成文件直写加校验循环：没有表单保护，换来的是任意 agent 可用与自动化友好。

两条路径当前的能力边界值得知道：

- **派生表**：`recommend` 与 `minister_assets` 由编辑器保存时自动生成，不经编辑器的目录若涉及举荐、立绘内容，需按[派生表说明](../reference/tables/derived-tables)自行处理；
- **图片资源**：编辑器有结构化槽位兜底，文件直写靠 `validate` 兜底——新增实体（不在原版同名表）的约定资源缺失会被 `asset-missing` 拦下（事件图、兵种图、技能图标、省份浮层、立绘四件套）；资源路径与 `Config/assets.json` 显式映射的完整约定见[图片与资源](../resources-and-ui/images-and-assets)，`preflight` 另拦注册映射与资源合法性错误。

两条路径殊途同归：产物是同一套 Mod 目录契约，发布前都过同一道 Workshop 整目录预检（见[发布门禁与 preflight](./preflight-and-publish)）。用编辑器做到一半的 Mod，也可以随时交给 agent 用工具链继续改，反之亦然。

## 质量标准与版本

`mod-toolkit/guide` 工作流契约为所有任务定义了统一的质量标准：**`validate` 0 error + `preflight` valid + `quality-gate` 未裁定 issue 为 0 且已裁定无 error + 缺表 0**。质量门可含 warning 和 error；三个工具各管一层——结构校验、发布门禁、语义启发式——完整语义见本篇后续章节。

包版本跟随游戏 workspace 发布；校验与预检报告携带 `schemaVersion`（Mod 数据 schema 版本），schema 演进会反映到包版本号。文档、原版数据与官方字段基线的查询根目录默认随包分发，也可用环境变量 `MOD_TOOLKIT_DOCS_DIR` / `MOD_TOOLKIT_VANILLA_DIR` / `MOD_TOOLKIT_FIELD_DEFINITIONS_DIR` 覆盖。
