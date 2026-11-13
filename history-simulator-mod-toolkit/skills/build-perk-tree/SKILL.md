---
name: build-perk-tree
description: "创作《历史模拟器：崇祯》Mod 的国策树节点（perks_table）——四个正式分类选型、Skill.NNNN 命名与改名落点同步、主前置/完整前置体系、互斥四规则双向维护、四类效果字段与两态图标。用户要求做一条国策树、加几个国策、让三个国策互斥、给国策配前置/效果/图标时使用；修既有国策的校验报错（perk-relation-invalid 等）走 repair-errors（修错），新增大臣/兵种/势力等非国策实体走 extend-tables（扩表），国策图标放图与命中核对走 manage-mod-assets（资源）。 English triggers: build a perk tree, add national policies, mutually exclusive perks, perk prerequisites and layout, perk effects and icons."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：创作国策树节点

触发：用户要求"做一条国策树 / 加几个国策 / 让几个国策互斥 / 配国策前置、效果或图标"——国策树（perks_table）节点的创作与关系维护：分类、命名、前置、互斥、效果、图标。

## 步骤

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。perk-relation-invalid 全部子规则修法详表、效果四类字段分工表、13 张可写表闭集、改名五处同步清单与图标键约定收在本技能 `references/perk-rules.md`——正文只留决策点，动笔写前置/互斥前先读它。拿不准分类与布局语义时调用 MCP `docs_page` 读 `game-content/perk-tree/` 下五篇。

1. **分类与命名**：category 正式分类只有四类——管理 / 军事 / 学识 / 政治（游戏端 Management / Military / Technology / Politics）。枚举里还有"随机开局"，但正式国策页面只提供四个分类入口、本地 Preview 把不能识别的分类映射到 other——需要玩家正常浏览和解锁的主路线不要放进去。code_name 保持 `Skill.NNNN` 四位编号形式（编辑器新增节点时扫描现有 `Skill.数字` 生成下一个编号），含义写进名称和描述，不要频繁改 ID；必须改时先跑 MCP `rename`（dryRun: true，只读）拿改名落点清单再逐处同步。
2. **动笔前查**：调用 `schema_table` 查 perks_table 的字段、必填与关系列语义；调用 `vanilla_table` 对照原版节点的分类分布、前置链长度与效果量级。
3. **前置体系**：`main_prerequisite` 是画布主要入边（决定主路径连线与高亮），`prerequisites` 才是解锁判定检查的完整前置列表——游戏端解锁要求完整列表每一项都已解锁，不只看画布主边。硬规则：主前置必须与当前国策同分类（副前置允许跨分类，但会把玩家引向另一棵树，需专项测试）；主前置必须同时包含在 prerequisites 中。建完前置做防不可达检查：前置 ID 都存在、无自引用与循环、所有正式节点最终能从某个根节点到达、副前置不要求进入无法访问的分类或互斥路线。`Skill.0000` 是"无前置"哨兵值（校验豁免），不要拿它表达业务条件。
4. **互斥**：互斥 = 选了 A 不能再选 B，与前置是两套独立约束。CSV 中必须双向保存（A 的 mutually_skill 含 B 且 B 的含 A），只写一端校验即报。四条禁令：禁与自身互斥；禁引用不存在的节点；禁与同一条前置路径上的节点互斥（逻辑矛盾——后继要求前置已完成，前置又禁止后继完成）；互斥双方必须同属一个正式分类（游戏端互斥索引只接受同分类关系，跨分类被静默忽略）。
5. **效果与图标**：四类字段分工——current_turn_effect（本回合结构化数据库效果，唯一真改数据的字段）、effect_display（玩家看到的效果说明）、context_text（本回合推演上下文）、long_term_prompt（后续回合长期语义）。不要用展示文字替代结构化效果，也不要在提示词里要求模型重算已被数据库效果应用的数值（重复影响）。current_turn_effect 只能写 13 张可写表闭集内的规范英文表名。每个国策配两态图标：`dynamic-assets/skill-icons/Skill.<数字>-lock.<扩展名>` 与 `Skill.<数字>-unlock.<扩展名>`——放图与命中核对全流程走 manage-mod-assets 技能；放完后调用 `asset_map` 确认请求键真实命中 overrides。
6. **validate 闭环**：每写一批调用 `validate`，perk-relation-invalid 按子规则修（逐条修法见 references）；拿不准调用 `codes` 查该码全部 reason 与修复指引。error 清零后再跑 `quality_gate` 与 `preflight`。
7. **验证边界交接**：Preview 能查分类映射、节点与前置边生成、名称/描述/效果文字与图标显示；但不输出 mutually_skill（互斥节点在 Preview 里仍可同时选择，不代表配置无效）、不执行 current_turn_effect、不持久化真实解锁、不应用长期提示词。互斥与真实解锁必须移交「打包版 + Workshop 测试场景」人工验收（七组用例清单见 references）。

## 高危踩坑（实战最高频坑，动工前通读）

- **改 code_name 漏一处同步就是连锁崩**：prerequisites、main_prerequisite、双向 mutually_skill、两态图标文件名、事件/决议的 skill_id 条件五处都引用同一 ID——改名前必须先跑 rename dry-run 拿全量落点清单，改完逐处核对，再跑 references 确认旧名零残留。
- **验收关系不验收坐标**：画布位置只存浏览器键（mod-maker:perk-positions），不进 perks_table.csv、不随 Mod 分发；游戏端按服务端返回的节点和边重跑 Dagre 布局。拖动节点不建立/删除前置，编辑器布局漂亮不等于玩家端布局相同——不要试图控制最终像素位置。
- **互斥只写单向是最高频 perk-relation-invalid**：手工改 CSV 时 A 写了 B、B 没写 A 即报错；互斥对必须两端同步维护，删除节点时也要从其他节点的 mutually_skill 里清掉该 ID。
- **"随机开局"不是主路线分类**：它在枚举里合法，但正式国策页面只有管理/军事/学识/政治四个入口，Preview 把它当 other——主路线放进去等于对玩家不可见。
- **主前置必须同分类且进完整前置列表**：跨分类只能做副前置；main_prerequisite 不在 prerequisites 里、或与当前国策不同分类，都是 perk-relation-invalid。

## 检查点

- [ ] 主路线节点全部落在四个正式分类（未用"随机开局"）
- [ ] code_name 保持 Skill.NNNN 形式；改名经 dry-run 落点清单完成五处同步（prerequisites / main_prerequisite / 双向 mutually_skill / 两态图标 / 事件决议 skill_id）
- [ ] 主前置与当前国策同分类且包含在 prerequisites 内；前置无自引用、无循环、不可达
- [ ] 互斥全部双向保存、同分类、不与前置路径冲突（无 perk-relation-invalid）
- [ ] current_turn_effect 只写 13 张可写表闭集；effect_display 与实际运算逐项对应；提示词不重复应用即时数值
- [ ] 两态图标按 Skill.<数字>-lock/unlock 键放置且覆盖索引真实命中
- [ ] Preview 未验证项（互斥生效 / 真实解锁 / 效果执行 / 长期提示词）已移交「打包版 + Workshop 测试场景」
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** the user asks to create national policy / perk tree nodes (perks_table) — categories, naming, prerequisites, mutual exclusion, effects, icons. Fixing validation errors on existing perks (perk-relation-invalid etc.) belongs to repair-errors; adding non-perk entities (ministers, troops, regimes) belongs to extend-tables; perk icon placement and hit verification belong to manage-mod-assets.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Category and naming: the four formal categories are 管理/Management, 军事/Military, 学识/Technology, 政治/Politics. "随机开局" is a legal enum value but the production perk page only exposes the four categories and Preview maps unknown categories to `other` — never put the main progression there. Keep code_name in the `Skill.NNNN` four-digit form; before any rename run `rename` with dryRun to enumerate every landing spot.
2. Look before writing: `schema_table` for perks_table fields and relation-column semantics; `vanilla_table` for official category distribution, prerequisite chain lengths, and effect magnitudes.
3. Prerequisites: `main_prerequisite` is the canvas main in-edge (routing/highlight); `prerequisites` is the complete list the unlock check evaluates — every entry must be unlocked. Main prerequisite must share the node's category (sub-prerequisites may cross categories) and must also appear in prerequisites. After wiring, verify: every prerequisite ID exists, no self-reference or cycles, all formal nodes reachable from a root, sub-prerequisites don't require inaccessible categories or mutually exclusive routes. `Skill.0000` is the exempt "no prerequisite" sentinel — do not use it for business logic.
4. Mutual exclusion: independent from prerequisites ("cannot both" vs "must first"). Save BOTH directions (A lists B and B lists A) — one-sided rows fail validation. Four bans: no self-exclusion; no referencing nonexistent nodes; no excluding a node on the same prerequisite path (logical contradiction); both nodes must share one formal category (the game-side exclusion index silently ignores cross-category pairs).
5. Effects and icons: four field roles — current_turn_effect (structured database effect, the only one that changes data), effect_display (player-facing text), context_text (this-turn inference context), long_term_prompt (long-term semantics). Never substitute display text for structured effects, and never ask the model to re-apply values already applied by database effects. Effects may only target the 13-table writable closed set with canonical English names. Two-state icons: `dynamic-assets/skill-icons/Skill.<digits>-lock.<ext>` and `-unlock` — full placement flow in manage-mod-assets; confirm real hits with `asset_map`.
6. Validation loop: `validate` after every batch; fix perk-relation-invalid per sub-rule (full table in `references/perk-rules.md`); `codes` for any code's full reason list; then `quality_gate` + `preflight`.
7. Verification boundary: Preview proves category mapping, node/edge generation, and name/description/effect-text/icon display only — it does NOT output mutually_skill (mutually exclusive nodes stay co-selectable in Preview; that does not invalidate the config), never executes current_turn_effect, never persists unlocks, never applies long-term prompts. Hand off to manual acceptance on the packaged build with a Workshop test scenario (seven test-case groups in references).

**Top pitfalls:** renaming a code_name without syncing all five reference sites (run rename dry-run first); canvas coordinates live in a browser key and never reach the CSV — the game re-runs Dagre layout, so accept relationships, not pixel positions; one-sided mutual exclusion rows; treating "随机开局" as a main-route category.
