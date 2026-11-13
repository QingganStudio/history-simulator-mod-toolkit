
# Skill：从零创建一个 Mod

触发：用户要求“从零做一个 mod / 新建一个 mod / 做一个 XX 主题的 mod”。

## 步骤

以下命令用于旧 CLI；非插件 MCP 使用同名的下划线工具（MCP 名 `mod_scaffold` 对应 `mod-scaffold`）。

1. 根据用户给出的时代、势力格局和目标选择路线。有现成场景基线时用 `baseline-get` 读取，世界观事实变化后用 `baseline-set` 保存。
2. 改良原版用 `mod-scaffold <空目录> --from-vanilla` fork 完整剧本；全新剧本用 `mod-scaffold <目录> --table <表名>... --prompt` 建骨架——完整骨架指全部 27 张源表（用 `schema-list` 取名单）加 prompt 层，需要示例层再加 `--example`。两种路线不要混用，增量补表时再次调用 `mod-scaffold --table`。骨架落成只代表结构齐备：setting 层仍带 skeleton-unchanged 警告，按 `skills/rewrite-prompt-layers.md` 把 setting 与 example 写成本剧本内容后才算世界观完成；rules 保持权威副本。
3. 写任何表前用 `schema-table` 查字段，用 `vanilla-table`、`vanilla-json` 或 `vanilla-prompt` 查参照。大臣和举荐源表变化后用 `derive-tables` 重建派生表；删改实体前用 `references` 查影响面。
4. 每批修改后运行 `validate`，收尾再运行 `quality-gate` 与 `preflight`，分别报告错误、缺表、质量警告与发布预检结果。

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
