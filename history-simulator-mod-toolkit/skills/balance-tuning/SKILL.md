---
name: balance-tuning
description: "调整《历史模拟器：崇祯》Mod 的已有数值（税率、兵种成本、事件强度、民心/叛乱等参数）。用户要求调平衡、改数值、调难度（太难/太简单）、对比原版量级调整时使用；新增表/实体/事件走 extend-tables（扩表），按错误报告修错走 repair-errors（修错）；用户要的是写脚本/批处理程序本身（而非本工具面内的调参工作流）时不适用。 English triggers: balance tuning, adjust values, tweak difficulty and economy, change tax rates, troop costs, event strength, popular support parameters."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：平衡调整

触发：用户要求调数值（税率、兵种成本、事件强度、民心/叛乱参数等）。

## 步骤

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。

1. **定位数值**：调用 MCP `schema_table` 确认字段所在表与列语义（类型、required、min/max）；用 `docs_search` 查该机制的官方说明（数值单位、结算时点）。
2. **取参照**：调用 `vanilla_table` 看原版量级（按行名精确读取目标行）；读 mod 现值，算出变化幅度。
3. **小步快跑**：用宿主编辑一次改一组相关字段（如一个势力的税+民心联动项），立刻调用 `validate`。数值列有 min/max，越界即 error。
4. **语义复核**：改完调用 `quality_gate`——收支失衡、民心-叛乱联动、驻地矛盾是它的主业。把报告分成"原版继承"与"本次新增"两堆：新增 warning 要么继续调整数值、要么向用户说明设计意图由作者裁定；不代为写 ack 豁免。
5. **给用户看 diff**：按"表.列：旧值 → 新值（理由与参照来源）"汇报；用户已给出明确目标或方向时直接执行、汇报即确认，不重复追问，只有二选一的设计判断才停下来问。交付前跑 `preflight` 确认可发布。

## 高危踩坑（实战最高频坑，动工前通读）

- **数值区间是三层分工，别只盯一层**：schema min/max（validate 报 error）管列定义硬边界；schema `advisoryMin`/`advisoryMax` 由 quality-gate 的 `numeric-range` 读出建议范围（0-100 / 0-10 / 0-1 分档）并叠加偏离基线告警；preflight 复用 schema 硬边界。调参后三层都要过，"validate 绿"不等于"数值合理"。
- **setting 世界观决定数值语义**：税率、民心、叛乱等参数的实际含义受 `prompt/setting/` 世界观描述影响（AI 按该文本解读数值语境）——setting 层还是原版崇祯内容时，你调的数值在游戏里被按崇祯语义推演，平衡结论会失真。

## 检查点

- [ ] 所有改动都在 schema min/max 内
- [ ] quality-gate 无新增未解释警告（继承警告与新增警告已分开报告）
- [ ] 改动有参照（原版量级或用户指定）
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** the user asks to tune existing numeric values (tax rates, troop costs, event strength, popular-support/rebellion parameters), often "make it easier/harder" or "match vanilla magnitude". Adding new content belongs to extend-tables; fixing error reports belongs to repair-errors.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Locate values: `schema_table` for the column (type, min/max); `docs_search` for the mechanic's official doc (units, settlement timing).
2. Reference: `vanilla_table` by row key for vanilla magnitude; read current mod values and compute the delta.
3. Small steps: change one related group of fields, run `validate` immediately (numeric columns have min/max — out of range is an error).
4. Semantic review: `quality_gate` (budget balance, support-rebellion coupling, garrison conflicts are its core). Split the report into "inherited from vanilla" vs "newly introduced"; new warnings either get tuned or explained to the author — never write acks on the author's behalf.
5. Report a diff: `table.column: old → new (reason + reference)`; when the user gave a clear direction, execute and report without re-asking. Run `preflight` before delivery.

**Top pitfalls:** three numeric layers (schema min/max via validate, semantic baseline via quality gate, upload gate via preflight) — all three must pass; the setting layer defines the semantic context of numbers — while setting still reads vanilla-Chongzhen, tuned values get interpreted under Chongzhen semantics.
