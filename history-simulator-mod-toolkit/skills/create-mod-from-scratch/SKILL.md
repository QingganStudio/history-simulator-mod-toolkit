---
name: create-mod-from-scratch
description: "创建《历史模拟器：崇祯》Mod。用户要求从官方原版 fork 改良 Mod、新建 Mod，或创建全新世界观剧本（含数据表与 prompt 层）时使用；已有 Mod 的局部增补交给 extend-tables（扩表），不动数据表、只换世界观文案的改写走 rewrite-prompt-layers。 English triggers: create a new mod, fork vanilla scenario, build scenario from scratch, start a mod project, new scenario, new world."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：从零创建一个 Mod

触发：用户要求“从零做一个 mod / 新建一个 mod / 做一个 XX 主题的 mod”。

## 步骤

插件需要 Node.js 20 或更新版本和宿主文件编辑能力。先将用户指定的 Mod 根目录转为绝对路径，保留已有的 `AGENTS.md`、`CLAUDE.md`；安装插件本身无需向 Mod 目录写文件。

1. 区分改良原版与全新剧本。改良原版时，在空目录调用 MCP `mod_scaffold`，传 `directory`、`fromVanilla: true` 和用户已给出的 `id`、`name`、`description`、`author`。`id` 只接受字母、数字、下划线和连字符，最长 64 字符；`fromVanilla` 与 `tables`、`prompt` 互斥。目录非空时向用户指出冲突并使用其指定的其他空目录。
2. 全新剧本时生成完整骨架：先调用 MCP `schema_list` 取全部 27 张源表名单，再调用 `mod_scaffold` 传 `directory`、全量 `tables`、`prompt: true`，需要示例层时加 `example: true`——完整骨架落成即 `validate` 缺表为零。骨架只是起点：prompt/setting 仍带 `skeleton-unchanged` 警告，按 rewrite-prompt-layers 技能把 setting 与 example 写成本剧本内容（rules 权威副本已由骨架携带，服务端不落库、不使用 rules 层，无需关注），并用宿主文件编辑能力逐表写内容。确定 Mod 内容前不要宣布完成。
3. 修改数据前调用 `schema_table` 查询字段，再用 `vanilla_table`、`vanilla_json` 或 `vanilla_prompt` 查原版参照。改动大臣或举荐源表后调用 `derive_tables` 重建对应派生表；改名或删除实体前调用 `references`。
4. 每批修改后调用 `validate`。完成时分别检查 `errorCount`、`contractCoverage.missing`、`quality_gate` 和 `preflight`；原版继承的 warning 与新增 warning 分开报告，需要作者判断的设计警告保持未裁定。

## 高危踩坑（实战最高频坑，动工前通读）

- **缺表必须清零**：`validate` 报告的 `contractCoverage.missing` 为空才算数据面齐备；缺表时游戏按空表处理。
- **setting 层必写世界观**：`ScenarioMeta.yaml` 声明的剧本中，`prompt/setting/` 文件须体现本剧本；骨架文件存在不代表世界观已完成（rules 层由骨架携带权威副本，服务端不使用、无需关注）。
- **recommend 是派生表**：`recommend` 由 `recommend_base` × `recommend_ministers` 联结生成；修改源表后重建，不要直接手写产物。
- **资源键沿用官方名称**：random 事件图把 code_name 中的点换为连字符；新兵种图使用 `client-assets/dynamic-assets/troops/<code_name>.webp`。

## 内容依赖链（分域施工顺序）

27 张源表骨架只是起点——内容按依赖链分层推进，本层四件套绿再进下一层。层序与层内顺序的权威表述见文档 `game-content/scenario-data/dependencies-and-identifiers`（7.1 节，`docs-page` 可读）；跨层引用的唯一纽带是稳定标识 code_name——FK 单元格、事件/决议/国策内联引用、GeoJSON 要素、资源键都吃同一个名字，任何改名先 `rename --dry-run` 拿落点清单。

- **基础层**（路由 extend-tables / balance-tuning）：政权/党派/社会阶层（regimes、parties、social_classes——上游字典）→ 大臣/官职/举荐（ministers、recommend_ministers、recommend_base 引用 parties；大臣两表 code_name 落定后跑 `derive-tables` 重建派生表；立绘四件套 `-chat`/`-face`/`-card`/`-argue` 的资源标识由 code_name 去数字前缀派生，随本层规划）→ 省份/土地/建筑/财政（land_tax_configs 先于 province_land_holdings，provinces 引用 regimes）→ 军事四表（equipments、arm_configs、ship_configs 是编制字典可随前两层建，troops 实际军队依赖政权归属与驻地省份，放在省份之后）→ 开局（玩家属性/初始任务/事件预测/特殊推演，主体稳定后配置）。
- **事件链**（路由 create-events）：六类事件表（常规/随机/固定回合/链式/超事件/决议）→ 事件图资源键（随机三表共用命名空间、点号换连字符）。事件条件与效果读写基础层各表——基础层未绿不动笔。
- **国策链**（路由 build-perk-tree）：perks_table 节点 ↔ 两态图标 ↔ 事件/决议的 skill_id 引用。与事件链并行推进，但 Skill.NNNN 命名先定，供事件链的条件引用。
- **世界层**：地图（GeoJSON 省份要素 ↔ provinces code_name ↔ 省份浮层 ↔ 视图配色 province-view-config，路由 custom-map）→ 资源（立绘/事件图/图标/底图的放图与命中核对——插件形态走 manage-mod-assets 技能，旧 CLI 形态见 AGENTS.md「校验错误怎么办」一节与文档 `resources-and-ui/images-and-assets/registered-images.md`；放好后用 `asset-map` 确认请求键真实命中）→ prompt 层（setting 世界观决定数值语义与 AI 推演口径，路由 rewrite-prompt-layers——别拖到最后，skeleton-unchanged 警告清零才算世界观完成）。
- **配置类穿插**：历法、官职体系、全局配置等注册 JSON 随对应层插入（main_regime_code 随政权、官职体系随大臣、历法随开局与事件预测）——插件形态走 edit-mod-config 技能，旧 CLI 形态见 AGENTS.md 校验错误分节与文档 `advanced/game-and-ui-config/` 各篇。

## 检查点

- [ ] 用户要求的内容已写入，骨架未被误认为成品（全新剧本的 setting 层 skeleton-unchanged 警告已清零）
- [ ] 内容按依赖链分层推进，本层四件套绿才进下一层（未跳层返工）
- [ ] 改名或删除涉及的引用已核对
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** the user asks to create a new mod, fork/improve the vanilla scenario (chongzhen-1627), or build a brand-new world from scratch. Incremental additions to an existing mod belong to extend-tables.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Route choice: improve-vanilla → `mod_scaffold` with `fromVanilla: true` in an EMPTY directory; brand-new world → `schema_list` for all source tables, then `mod_scaffold` with full `tables` + `prompt: true` (+ `example: true` for example layers). The two routes are mutually exclusive.
2. A generated skeleton is a starting point, not a finished mod: setting layers still carry `skeleton-unchanged` warnings until you rewrite them (see rewrite-prompt-layers). Do not declare completion before content is written.
3. Before writing any table: `schema_table` for fields, `vanilla_table` / `vanilla_json` / `vanilla_prompt` for official reference. After touching ministers/recommend source tables, run `derive_tables`. Before renaming/deleting an entity, run `references`.
4. After every batch of edits run `validate`. At completion report the delivery four-piece separately: validate 0 error / missing tables 0 (`contractCoverage.missing` empty) / preflight valid / quality gate with inherited vs new issues separated (never ack on the author's behalf).

**Dependency chain (build order):** the 27-table skeleton is only the start — content advances in layers, each gated on the delivery four-piece before the next. Foundation (regimes/parties/social classes → ministers/positions/recommendations, rebuilding derived tables after code_name changes → provinces/land/buildings/finances → military dictionaries then actual troops → opening data) routes to extend-tables / balance-tuning; events (six tables → image keys) to create-events; perks (perks_table ↔ two-state icons ↔ skill_id references) to build-perk-tree; the world layer (map GeoJSON ↔ provinces ↔ overlays ↔ view colors → assets → prompt layers, whose setting text decides value semantics — do not leave it last) routes to custom-map / manage-mod-assets (plugin form) / rewrite-prompt-layers. Layer order authority: docs `game-content/scenario-data/dependencies-and-identifiers` (7.1); code_name is the single cross-layer identifier — `rename` dry-run before any rename.

**Top pitfalls:** missing tables are silently treated as empty by the game — `contractCoverage.missing` must be empty; `recommend` is a derived table (rebuild, never hand-write); random-event images replace dots with hyphens in the asset key.
