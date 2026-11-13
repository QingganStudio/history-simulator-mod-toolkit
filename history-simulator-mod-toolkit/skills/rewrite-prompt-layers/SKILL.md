---
name: rewrite-prompt-layers
description: "填写或重写《历史模拟器：崇祯》Mod 的 prompt 文案层（setting 世界观、example 示例）。用户要换皮、换世界观文案、改写 prompt 设定，或骨架生成后填充 prompt/setting 与 prompt/example 时使用——不建表、不动 CSV 数据；从零搭建整个 Mod（含数据表）走 create-mod-from-scratch。rules 层只保留脚手架携带的权威副本，永不手改。 English triggers: reskin prompts, rewrite setting layer, world-building text, new scenario setting, fill prompt skeleton, change world lore."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：重写 prompt 三层（新建剧本/换皮）

触发：用户新建剧本 mod、要求"换皮/换世界观/改 prompt 层"，或 scaffold 后要填充 prompt/ 目录内容。

## 步骤

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。全新剧本的骨架生成走 create-mod-from-scratch 技能；本技能负责把骨架的 prompt 层写成本剧本内容——`validate` 报告里 prompt-layer 码的 `skeleton-unchanged` 警告全部清零前，世界观不算写完，不能因骨架生成成功就宣布完成。

1. **查契约面**：调用 MCP `validate`（传 `directory`），读报告的 `contractCoverage.promptLayers` 槽位视图（14 条目 × 层：source=mod/absent/no-slot、字符预算、变量解析结论），确认内容面进度与缺口。
2. **取参照**：逐名调用 MCP `vanilla_prompt`（传 `name`，按需要用 `layer` 取 `setting` 或 `example`）取原版原文与元信息（坑位提示先读——NPCUnifiedPrompt 是 LLM 前缀缓存键、db_info 外键字段要写 _code_name 别名（regime/location_province/party 三对），不抄数据层的 _id 列名、CourtDiscussionPrompt 是唯一 jinja 渲染条目）。
3. **rules 层无需关注**：服务端不落库、不使用 rules 文件（仅展示），本地校验也不再覆盖其内容——`mod_scaffold` 以 `prompt: true` 携带的权威副本保持原样即可，无需核对、无需改写。
4. **重写 setting 层**（10 个全改，不是子集）：每文件换成你的世界观/人物/纪年/字段语义；对无 setting 槽的已知名可开新槽（`mod_scaffold` 传 `prompt` 名单如 `["AgentLoop-Simulate-Step1"]`，sanguo 实证）。static 条目里 `{{ }}` 不会渲染、会原样发给模型——别写；改写 CourtDiscussionPrompt 时保留其 11 个业务变量占位。**字段口径**：db_info 及一切含数据库字段引用的 setting/example 文本，外键字段一律写 `_code_name` 别名（`regime_id→regime_code_name`、`location_province_id→location_province_code_name`、`party_id→party_code_name`），与服务端工具契约参数同名以提高 function call 命中率——不要照抄 schema-table / CSV 表头的 `_id` 列名；`_id` 只用于 CSV 与事件条件 JSON。
5. **重写 example 层**（换皮必改；`mod_scaffold` 传 `example: true` 生成 6 槽骨架）：保留原版分节骨架（评分体系/章节结构/输出格式），叙事整篇换皮。
6. **ScenarioMeta 增键不删键**：称呼/纪年/术语改键（edict_term/era_name/display_terms.*——模板已引用，改键即生效）；新键需确认后端有消费方（scenario-meta-unknown-key warning 是提示不是错误）。
7. **闭环**：`validate` 与 `preflight` 全绿——prompt-layer(skeleton-unchanged) / prompt-render-residual / scenario-meta-schema 逐码清零；再跑 `quality_gate`，把仍需作者判断的设计警告逐条报告作者，不代为裁定。

## 高危踩坑（实战最高频坑，动工前通读）

- **rules 层与校验脱钩**：rules 文件服务端不落库、不使用（仅展示），本地 validate/preflight 也不再检查其内容——缺不缺、改没改都不会报错；机制语义差异一律放 setting/example 层表达。
- **static 条目写 `{{ }}` 不渲染**：12 个 static 条目渲染失败时 fail-open——未渲染原文（含花括号）会原样发给模型；只有 CourtDiscussionPrompt 走严格渲染（拼错即崩）。变量宇宙 = ScenarioMeta 顶层键 + time_label/time_note（廷议另有 11 个调用点变量），拿不准就带 `| default("...")`。
- **prompt 层写 `_id` 列名是静默劣化**：把 `regime_id` 等数据层真实列名写进 db_info / setting / example，校验不报任何错误，但模型的 function call 命中率会下降——prompt 层字段口径只认 `_code_name` 别名（见步骤 4）。

## 检查点

- [ ] 14 条目槽位视图里 5 个 fatal 名（三阶段四件套 + EventManagementAgent）的层齐备
- [ ] setting 无 skeleton-unchanged 残留；渲染模拟零残留
- [ ] preflight valid（prompt 族 7 码全绿）
- [ ] quality-gate 已跑；剩余设计警告已逐条报告作者且未代为裁定
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** filling or rewriting the mod's prompt layers — new world/scenario, reskin, or populating the prompt/setting and prompt/example layers after scaffolding. The rules layer needs no attention (the server does not consume it; scaffold-carried copies are fine as-is).

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Contract face: `validate` → `contractCoverage.promptLayers` slot view (entries × layer: source=mod/absent/no-slot, char budget, variable resolution) — the world is not "done" until every `skeleton-unchanged` warning is cleared.
2. Reference: `vanilla_prompt` per logical name (+ `layer`) — read the hint first (NPCUnifiedPrompt is an LLM prefix-cache key; db_info foreign keys use `_code_name` aliases — regime/location_province/party — never the data-layer `_id` column names; CourtDiscussionPrompt is the only jinja-rendered entry).
3. rules layer: no action needed — the server neither stores nor uses rules files (display only), and local checks no longer cover their content; the authoritative copies carried by `mod_scaffold` `prompt: true` are fine as-is.
4. setting layer: rewrite ALL files (not a subset) into your world; new slots for known names go through `mod_scaffold` with a `prompt` list. No `{{ }}` in static entries (sent verbatim); keep CourtDiscussionPrompt's 11 business variable placeholders.
5. example layer: keep the official section skeleton (scoring/chapter/output format), reskin the narrative; generated via `example: true`.
6. ScenarioMeta: change keys, never delete (edict_term / era_name / display_terms.*).
7. Close the loop: `validate` + `preflight` green on all prompt-family codes, then `quality_gate` with remaining design warnings reported to the author.

**Top pitfalls:** rules files are decoupled from validation (server does not store/use them; local checks ignore them) — put mechanism semantics in setting/example instead.
