---
title: 文本、对话与 AI 提示词
description: 编辑开局文本、13 个分层逻辑提示词与 1 个 YAML 剧本元信息
---

# 文本、对话与 AI 提示词

> 适用范围：当前 13 个 Markdown 逻辑提示词（按 setting/rules/example 三层组织）、1 个 YAML 剧本元信息（ScenarioMeta）与 1 个 TXT 文本契约；最近代码核对：2026-10-09；验证层级：编辑器格式预览 + Workshop 服务端场景回归。

## 开局文本

文本编辑器维护 `default_turn_0` 对应的 TXT 开场内容。它是首回合展示文本，不是提示词模板。保存前检查换行、标点和总字符数；TXT、每个 Markdown 层文件与 ScenarioMeta YAML 都不得超过 20,000 Unicode code points。

信息感知规则 `content` 以 `file:` 前缀引用的推演指令长文（`gsr_yuan_kills_mao`、`gsr_jisi_incident`）是官方内置内容，由 vanilla 剧本包携带：mod 不包含这两个文件，文本编辑器不展示它们，保存时也不会写入（存量 mod 目录里残留的旧副本会在保存时自动移除）。

## 大臣聊天和廷议

这两个客户端 JSON 编辑器（大臣聊天预设、廷议快捷语）已移至高级配置章的[13.10 大臣聊天预设](./game-and-ui-config/minister-chat-presets)与[13.11 廷议快捷语](./game-and-ui-config/court-chat-quick-messages)专门页；本页只保留分层提示词与 TXT 文本契约。

## 14 个逻辑提示词（13 个 Markdown 逻辑名 + 1 个 YAML）

每个逻辑提示词由一至三个**层文件**组成，磁盘路径为 `prompt/{setting|rules|example}/<逻辑名>.md`；`ScenarioMeta` 是唯一的 YAML，整体解析、不分层。

| logical name | 层组合 | 主要用途 |
|---|---|---|
| `ScenarioMeta` | 不分层（`ScenarioMeta.yaml`） | 剧本元信息（YAML） |
| `AgentLoop-Simulate-System` | 设定 + 规则 + 示例 | 推演系统规则 |
| `AgentLoop-Simulate-Step1` | 规则 + 示例 | 推演第一阶段 |
| `AgentLoop-Simulate-Step2` | 规则 | 推演第二阶段 |
| `AgentLoop-Simulate-Step3` | 设定 + 规则 + 示例 | 推演第三阶段 |
| `OutputGenerationAgent` | 设定 + 规则 + 示例 | 奏折等输出生成 |
| `EdictRefineAgent` | 设定 + 规则 | 诏书润色 |
| `EndGameAgent` | 设定 + 规则 + 示例 | 终局评价 |
| `TaskAgent` | 设定 + 规则 + 示例 | 任务生成/处理 |
| `EventManagementAgent` | 设定 + 规则 | 事件压缩与管理 |
| `NPCUnifiedPrompt` | 设定 + 规则 | NPC 对话 |
| `CourtDiscussionPrompt` | 设定 + 规则 | 廷议对话 |
| `EasyModeContent` | 规则 | 辅助模式内容 |
| `db_info` | 设定 | 数据库信息提示 |

三层职责：

- **设定层（setting）**：世界观、人物、语气，创作者可自由编辑；
- **规则层（rules）**：机制、协议与禁令，平台统一维护；服务端不落库、不使用 rules 层文件（仅供展示），本地与服务端均不再校验其内容。编辑器中只读展示，保存时自动携带编辑器内置的最新官方字节，无需改动磁盘上的规则层文件；
- **示例层（example）**：可选 few-shot，可清空；空示例层保存时省略写出、Manifest 不登记。

运行时按 设定→规则→示例 顺序把层文件**合成**为一份文本喂给模型；Mod 包内只保存原料，合成发生在服务端。BYOK（自带密钥）模式下同样如此：对话改走你配置的模型，但 prompt 组装仍在服务端完成，Mod 的 setting 层覆盖照常生效。Manifest 用键 `<逻辑名>.<层>` 登记每个层文件，并带 `layer` 与 `required`（示例层为 `false`）字段。已退役的 `v2_tools` 已从编辑器与契约中移除，不再引导编辑。

使用提示词编辑器的 Markdown 预览检查层级、列表和代码片段。预览只能检查渲染，不会证明模型行为正确。

![提示词编辑器并排显示 Markdown 源文与渲染预览](/screenshots/stage-4/21-prompt-markdown-preview.webp)

*图：Markdown 预览只验证格式和可读性；真实模型行为仍需在服务端场景中回归。*

游戏预览窗口不会把 `prompt/` 下的层文件与 `prompt/ScenarioMeta.yaml` 发送给模型，也不会把 `text/default_turn_0.txt` 注入开局流程。两类文件只能在编辑器中检查格式、变量、字符数和保存结果；本地导入同样不会使官方服务端读取这些文件。

## 变量与结构

保留编辑器和原版模板中约定的变量占位符、输出格式和结构边界。不要翻译变量名，不要把要求 JSON 输出的段落改成自由文本。工具调用的字段契约由服务端 Harness 统一生成，不属于 Mod 内容，层文件中没有需要创作者保留的工具字段。新增说明时优先在不改变机器可解析结构的位置补充。

## 安全修改流程

1. 备份当前可用版本。
2. 一次只修改一个目标，例如语气、长度或背景知识。
3. 保留变量和输出 Schema，使用 Markdown 预览检查格式。
4. 保存并通过 20,000 字符、UTF-8 和 Manifest 注册检查。
5. Workshop 服务端场景就绪后，用固定的多组局面做真实推演，对比修改前后的稳定性。
6. 检查幻觉、循环、格式破坏、过长输出和不应暴露的内部指令。

提示词修改可能改变游戏结果、成本和响应时间。不要只测试一次成功样例，也不要把 Mod Maker 预览 Mock 当成真实模型回归。

> **逐名参考已升级为契约生成**：14 个逻辑 prompt 的层组合/渲染形态/消费分级/变量表/坑位提示见 [Prompt 逐名参考总览](/reference/prompts/index)，创作方法见[三层 prompt 分工写作法](/advanced/prompt-reskin)。

下一步：[保存与内容预检](/validate-and-publish/save-and-preflight)。
