---
name: repair-errors
description: "修复《历史模拟器：崇祯》Mod 的校验与发布问题。用户提交 validate 或上传预检报告、要求按错误码修到通过，或 Mod 校验失败需要定位原因时使用；新建 Mod、扩表加内容、调整数值等创作请求分别走 create-mod-from-scratch（创建）/ extend-tables（扩表）/ balance-tuning（平衡调整）。 English triggers: fix validation errors, repair preflight report, pass checks, preflight fails, upload rejected, validation report."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：修复 Mod 错误

触发：用户拿着校验报告（validate/preflight 输出）要求修错；或要求"让这个 mod 通过校验"。其他症状先按下方「症状分流」表归口——游戏内崩溃/运行时行为异常不在 agent 静态能力面，不要错承诺。

## 步骤

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。

**症状分流**（先归口再动手——错出口比慢修复更糟）：

| 症状 | 归属 |
|---|---|
| 校验/预检报告（validate / preflight 的 issue 清单） | 本技能按码修法，走下方流程 |
| 资源"放进去了但没生效" | 用 `asset_map` 核对请求键真实命中——索引未命中会静默回退官方资源，"页面正常显示"不算数；全流程见 manage-mod-assets 技能 |
| prompt 文案/世界观不生效 | rewrite-prompt-layers 技能 |
| 游戏内崩溃/运行时行为异常 | 不在 agent 静态能力面——本地 `local_` Mod 恒走官方场景、预览 mock 不能证明真实触发（口径见文档 validate-and-publish/preview.md「预览明确不能证明什么」）；移交「打包版 + Workshop 测试场景」人工排查 |

1. **建立基线**：调用 MCP `validate`（传 `directory`）拿全量 issue 清单，按 `reasonCode` 分组。若用户带来的是上传被拒报告，改调 `preflight` 从其 issue 清单开工（分组修法同样适用）。这份清单就是工作清单——不要凭感觉改。
2. **按码定位修法**：对每个 reasonCode 调用 MCP `codes`（传 `code`）查该码的全部 reason 与修复指引（title/rule/params）。reasonCode 稳定、中文 message 措辞会演进——按码修，不按 message 猜，也不 grep 源码。
3. **逐类修复**（高频错误码 → 处理方式）：
   - `primary-key-missing/duplicate`：补主键或去重；主键是表的 `primary_key` 列。
   - `foreign-key-missing`：两种修法——引用值打错（改成目标表存在的值）或目标缺失（去目标表补一行，用 `scaffold` 起步）。改谁取决于设计意图，拿不准就问用户。
   - `invalid-enum` / `below-minimum` / `above-maximum`：查 `schema_table` 的合法值/范围后修正。
   - `event-key-duplicate`：同一命名空间（随机事件三表）code_name 冲突，重命名其一后用 `references` 追旧名引用。
   - `perk-relation-invalid`：前置/互斥关系破坏（自引用、循环、跨分类、非双向）——按 message 提示修。
   - `fixed-turn-event-invalid`：cleaned_condition 需要首个 `type:'turn'`、`judge_type:'equal'`、整数值。
   - 导入层 `csv-import`（missing-table / table-misplaced / 元数据块与表头类）：缺表用 `mod_scaffold` 传 `tables` 补建并登记；放错目录的表移回 `expected` 指向的归属路径。
   - 预检独有面（如 `asset-extension-unsupported` 扩展名白名单）：`validate` 不报、`preflight` 才拦——上传被拒清单按 `preflight` 输出修。
4. **删除类操作前必查** `references`：删一行可能留下多处悬空引用，先改引用方再删；改动大臣/举荐源表后调用 `derive_tables` 重建派生表。
5. **回归确认**：全部修复后再跑 `validate`、`preflight`、`quality_gate` 并确认 `contractCoverage.missing` 为空——四项结论分开报告；对比前后两份报告确认没有引入新 error。
6. **报告**：按"错误码 → 修了什么 → 结果"向用户汇报。

## 高危踩坑（实战最高频坑，动工前通读）

- **按 reasonCode 分派，不要按 message 文本猜**：message 会随实现演进措辞，reasonCode 稳定；拿不准某码的全部 reason 与修法就查 `codes --code <码>`，不要 grep 源码。例外（两个结构校验不带 reasonCode）：finances 单例多行、province_land_holdings 派生不一致，按 message 处理。
- **text-char-limit/approaching-limit 是余量预警不是格式错**：注册 md/txt/yaml 超 18,000 code points（20,000 硬顶的 90%）触发——修法是精简内容或拆文件，不是改编码；approaching 阶段就动手，别等撞上传墙返工。
- **导入类 warning 不是噪音**：`csv-import` 的 missing-table / table-misplaced 意味着游戏在按空表跑——修错清单要把 `contractCoverage.missing` 清空纳入完成定义。
- **校验与预检是两个面**：扩展名白名单、资源大小/像素预算等只在 preflight 报——只在 validate 绿就停手会把上传被拒留到发布前最后一刻；修错收尾两面都要复跑。

## 检查点

- [ ] 初始清单上的 error 全部清零
- [ ] 没有引入新 error（对比前后两份报告）
- [ ] preflight valid（上传门禁零容忍）
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** the user brings a validate/preflight report (or an upload rejection) and wants errors fixed to green. Creative work (new mod, adding content, tuning numbers) does not use this skill.

**Triage first:** asset overrides not taking effect → verify real hits with `asset_map`; prompt/setting text not taking effect → rewrite-prompt-layers; in-game crashes or runtime misbehavior are beyond the agent's static surface — local `local_` mods always run the official server scenario and preview mocks cannot prove real triggers, so hand off to manual verification on the packaged build with a Workshop test scenario.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Baseline: `validate` for the full issue list (or `preflight` when the user was rejected at upload); group issues by `reasonCode` — this list IS the work list, do not guess.
2. For each reasonCode look up `codes` with `code` for all reasons and the fix guidance (title/rule/params). reasonCodes are stable; message wording evolves — fix by code, never by message text, never grep the source.
3. Common fixes: primary-key-missing/duplicate → fix/deduplicate the PK; foreign-key-missing → correct the value or add the missing target row (use `scaffold`); invalid-enum / below-minimum / above-maximum → check `schema_table`; event-key-duplicate → rename one side and chase old references with `references`; csv-import missing-table → `mod_scaffold` `tables`; extension/size/pixel limits are preflight-only faces.
4. Before any deletion run `references`; after ministers/recommend changes run `derive_tables`.
5. Regression: re-run `validate` + `preflight` + `quality_gate`, confirm `contractCoverage.missing` empty and no NEW errors vs the before-report; report per the delivery four-piece.

**Top pitfalls:** fix by reasonCode, not message; text-char-limit warnings mean content over 18,000 code points (20,000 hard cap) — shorten or split, don't re-encode; validate green ≠ uploadable (preflight-only faces exist).
