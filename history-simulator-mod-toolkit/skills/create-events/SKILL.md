---
name: create-events
description: "创作《历史模拟器：崇祯》Mod 的事件与决议——常规/随机/固定回合/链式随机/超事件/决议六类的选型、命名空间、条件、选项效果与结构。用户要求写一条事件、加个决议、做链式事件链、配事件触发条件或效果时使用；修既有事件的校验报错走 repair-errors（修错），新增大臣/兵种/势力等非事件实体走 extend-tables（扩表），事件配图与资源覆盖走 manage-mod-assets（资源）。 English triggers: create events, add a decision, write a random event, event chain, fixed-turn event, super event, event conditions and effects."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：创作事件与决议

触发：用户要求"写一条…事件 / 加个决议 / 做链式事件"——六类事件（常规 normal_events、随机 random_events、固定回合 fixed_turn_events、链式 random_events_chain、超事件 super_events、决议 decisions）的新增创作。

## 步骤

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。条件 8 类型全表、13 张可写表闭集、选型矩阵详表与六个专属错误码修法详表收在本技能 `references/event-contracts.md`——正文只留决策点，动笔写条件/效果前先读它。

1. **选型**：按触发方式选表——常规历史/推演事件→normal_events；条件满足后参与随机抽取→random_events；指定回合触发→fixed_turn_events；选项进入下一节点→random_events_chain；独立大画面展示重大事件→super_events；玩家主动执行并记录次数→decisions。反模式：固定回合需求做成普通随机事件、连续剧情拆成互不关联的随机事件——后续验证会因此变难。拿不准调用 MCP `docs_page` 读 `game-content/events-and-decisions/event-types-and-keys.md` 的选型矩阵。
2. **定标识**：先查重再落笔——random_events / fixed_turn_events / random_events_chain 三表共用一个 code_name 命名空间（撞名即 event-key-duplicate），且三表共用 `client-assets/dynamic-assets/events/random/` 图片目录、文件名按点号→连字符换算（`random.tax.001` 与 `random-tax-001` 解析到同一张图，连字符形态也视为已占用）。复制既有事件后第一件事改 code_name。按剧情族命名：`random.tax_shortage.001` / `chain.border_crisis.1` / `decision.reform_grain_tax` / `super.capital_falls`。
3. **写条件与效果**：先调用 `schema_table` 确认目标表/字段存在，用 `vanilla_table` 对照原版同类事件的量级与措辞。三条硬规则：①精确指向 finances 单行表必须省略行字段（单目标省 row_name，比较条件两侧分别省 row_name_1 / row_name_2——空串、null、"Value" 都是非法占位）；②固定回合事件的 turn 条件只能是 `{ "type": "turn", "judge_type": "equal", "value": 整数 }`；③条件读取与效果写入只能命中 13 张可写表闭集的规范英文表名（闭集清单见 references）。选项三段式（c1_content 玩家文字 / c1_display 结果展示 / cleaned_c1_effect 机器效果）已填的组必须三段齐填，空选项组可以不配置。
4. **结构类专属**：链式——链前缀 = code_name 最后一个 `.` 之前部分（同前缀成链），后继是选项效果数组里的一个 follow_up 项，每选项最多一个；删除节点/整链后按清单核对：残留 follow_up、孤立节点、链首有效触发条件、其他类型事件/决议/提示词中的人工引用、被删节点的图片。决议——repeat_policy 有四种正式值 once / once_per_turn / cooldown / repeatable；用 schema_table 查子契约，cooldown 必填正整数 cooldown_turns，次数上限用 max_per_turn / max_total，留空不限制；visibility_conditions（不满足不显示）与 execution_conditions（不满足可显示不可执行）分工不同，不要机械复制。超事件——决议的 super_event 保存快照（event_id/title/description/image_url），源记录改标题/描述/图片后快照不自动更新；源被删应重选或清空。
5. **validate 闭环**：每写一批调用 `validate`，六个事件专属码按码修：event-key-duplicate→重命名其一并用 `references` 追旧名引用（follow_up / decision_id / event_id / 快照 / 图片文件名 / 提示词）；fixed-turn-event-invalid→补首个 turn + equal + 整数的顶层条件；super-event-snapshot-invalid→快照行恰为 code_name/title/description/cleaned_condition 四字段，删多余字段；decision-structure-invalid→按 message 的 column 定位（repeat_policy 闭集、条件树仅 all）；runtime-table-access-forbidden→改用 13 表闭集内的规范表名；forbidden-singleton-row-selector→删 finances 目标的行选择器字段。某码拿不准调用 `codes` 查该码全部 reason 与修复指引。error 清零后再跑 `quality_gate` 与 `preflight`。
6. **事件图衔接**：常规事件图 `dynamic-assets/events/normal/<code_name>.webp`；随机/固定回合/链式 `dynamic-assets/events/random/<点号换连字符>.webp`；超事件 `dynamic-assets/events/super-events/<code_name>.<扩展名>`。新增事件缺约定图会报 asset-missing——放图与命中核对全流程走 manage-mod-assets 技能；放完后调用 `asset_map` 确认请求键真实命中 overrides。
7. **验证边界交接**：Preview 能查标题/描述/分类/排序、选项数量与展示文字、决议 once 完成态；不能证明条件求值、抽取概率、数据库持久化、回合触发、链式后继、长期提示词与存档恢复——fixed_turn_events 不进 Preview 快照，random_events_chain 进快照但无页面适配器。未验证项移交「打包版 + Workshop 测试场景」人工验收（七组用例清单见 references）。

## 高危踩坑（实战最高频坑，动工前通读）

- **随机三表撞 key 是最高频返工**：random_events / fixed_turn_events / random_events_chain 共用 code_name 命名空间，图片目录还把点号拉平成连字符——`random.tax.001` 与 `random-tax-001` 抢同一张图；复制事件行不改 code_name 直接 event-key-duplicate。
- **finances 行字段必须消失而不是留空**：指向 finances 时 row_name（比较条件的 row_name_1 / row_name_2）要整字段省略——空串、null、"Value" 都是 forbidden-singleton-row-selector，内容预检直接拒。
- **决议超事件快照不跟随源记录**：super_event 保存的是选择当时的标题/描述/图片快照——源超事件改完后决议里仍是旧内容；保存前对照 super_events 同步，源被删就重选或清空，别留无源快照。
- **Preview 绿不等于会触发**：预览不求值条件、不抽概率、不走回合，fixed_turn_events 根本不进快照——"预览里看到了"只证明结构与展示，触发证据只能在 Workshop 服务端场景拿。
- **条件/效果的表名是 13 表硬闭集**：写中文名、简写或闭集外表（事件表自身、perks_table 都不在闭集内）即 runtime-table-access-forbidden——保存和上传硬失败，旧 Mod 不自动迁移、不豁免。

## 检查点

- [ ] 选型与触发方式匹配（固定回合没做成随机、连续剧情没拆散成无关随机事件）
- [ ] code_name 在三表共用命名空间唯一，点号→连字符换算形态也不撞（无 event-key-duplicate）
- [ ] finances 目标全部省略行字段；固定回合条件为 turn + equal + 整数
- [ ] 条件/效果的表名全部在 13 张可写表闭集内（无 runtime-table-access-forbidden）
- [ ] 决议 super_event 快照与源记录一致；链式删除后无残留 follow_up 与孤立节点
- [ ] 事件图按命名空间放置且覆盖索引真实命中（新增事件无 asset-missing）
- [ ] Preview 未验证项（条件求值/概率/回合触发/链式后继）已移交「打包版 + Workshop 测试场景」
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** the user asks to create events or decisions — normal / random / fixed-turn / chained-random / super events or decisions (the six event-domain tables). Fixing validation errors on existing events belongs to repair-errors; adding non-event entities (ministers, troops, regimes) belongs to extend-tables; event art and asset overrides belong to manage-mod-assets.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Pick the table by trigger style: normal_events (plain record), random_events (condition-gated random draw), fixed_turn_events (exact turn), random_events_chain (options lead to next node), super_events (full-screen milestone), decisions (player-initiated, execution counted). Anti-patterns: a fixed-turn need built as a random event, or a continuous story split into unrelated random events.
2. Name before writing: random_events / fixed_turn_events / random_events_chain share ONE code_name namespace (event-key-duplicate on collision) and one image directory (`client-assets/dynamic-assets/events/random/`, dots flattened to hyphens — the hyphenated form counts as taken too). After copying an existing event, change code_name first. Name by story family (`random.tax_shortage.001`, `chain.border_crisis.1`, `decision.reform_grain_tax`, `super.capital_falls`).
3. Conditions and effects: `schema_table` to confirm target table/fields, `vanilla_table` for official magnitude and wording. Hard rules: finances targets omit the row selector fields entirely; fixed-turn condition is exactly `{type:'turn', judge_type:'equal', value:<int>}`; reads/writes may only touch the 13-table closed set with canonical English table names. Each used option group needs all three parts (c1_content / c1_display / cleaned_c1_effect). Full tables live in `references/event-contracts.md`.
4. Structure-specifics: chain prefix = code_name up to the last `.`; one follow_up per option; after deleting chain nodes check stale follow_up, orphaned nodes, chain-head conditions, manual references, orphaned images. Decisions: repeat_policy is a closed set (`once` / `once_per_turn` / `cooldown` / `repeatable`); query schema_table for positive cooldown_turns/max_per_turn/max_total parameters, null/omitted caps are unlimited; visibility vs execution conditions are not interchangeable. Super-event snapshots do NOT follow source edits — resync or clear.
5. Validation loop: `validate` after every batch; fix the six event codes by code (rename + `references` chase for duplicates; turn+equal for fixed-turn; exactly four snapshot fields; decision structure per column; 13-table whitelist; omit finances row selector). `codes` for any code's full reason list, then `quality_gate` + `preflight`.
6. Event art keys: normal `events/normal/<code_name>.webp`, random/fixed/chain `events/random/<dots→hyphens>.webp`, super `events/super-events/<code_name>.<ext>`; missing art raises asset-missing — place files and confirm real hits with `asset_map` (full flow in manage-mod-assets).
7. Verification boundary: Preview proves titles/descriptions/sorting/option display/repeat-policy/count/cooldown simulation only — never condition evaluation, probability, turn firing, chain successors, persistence, or save restore. Hand off to manual acceptance on the packaged build with a Workshop test scenario (seven test-case groups in references).

**Top pitfalls:** the shared three-table namespace flattens dots to hyphens for images (copy-paste without renaming = duplicate key); finances row selectors must be omitted, not blanked; decision super-event snapshots go stale silently; Preview green never proves real triggering.
