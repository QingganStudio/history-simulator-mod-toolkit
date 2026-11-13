
# Skill：扩表现有 Mod

触发：用户要求"加几张表 / 补充事件 / 增加 XX 内容"到现有 mod。

## 步骤

以下命令用于旧 CLI；非插件 MCP 使用同名的下划线工具（MCP 名 `list_tables` 对应 `list-tables`）。

1. **先看现状**：`list-tables <目录>` 了解已有数据；读相关表的现有行，理解作者的命名与数值风格。
2. **查规范与参照**：`schema-table` + `vanilla-table`（原版同表的量级和措辞是最好的风格样本）。
3. **增量写入**：新行追加在数据区末尾，保持列结构与元数据块不动。逐表小批量写，不要一次性生成全部再校验。尚不存在的表用 `mod-scaffold <目录> --table <表名>` 增量建表并登记 manifest，不要手建 CSV。
4. **每批 validate**：error 清零再写下一批。新增行引用了其他表的实体时，先确认目标存在（`vanilla-table` 或读目标表）。
5. **派生与引用**：改动大臣或举荐源表后跑 `derive-tables` 重建派生表；新增实体缺约定资源会报 asset-missing（按报错给出的请求键补 `client-assets` 文件）。改名/删除前跑 `references` 查影响面、`rename --dry-run` 拿落点清单，先改引用方再动定义行。
6. **一致性收尾**：`quality-gate` 看语义警告；交付前跑 `preflight` 确认 valid（扩数据一般不触发它，但它定义可交付）；如改动影响世界观事实（新势力/新大臣），更新 `baseline-set`。

## 高危踩坑（实战最高频坑，动工前通读）

- **扩表先核缺表**：开工先看 `validate` 的 `contractCoverage.missing` 与 list-tables 输出——以为"表已存在"而实际缺失时，后续所有对该表的写入都在给一张游戏按空表处理的幽灵表填数据。
- **注册文本有 20,000 码点硬顶**：`prompt/`、`text/` 下注册的 md/txt/yaml 超 20,000 code points 会被上传门禁拦（超 18,000 即预警）。照文档/原版内容改写前先看 `docs-page` / `schema-table` 输出附带的 charCount/charLimit 余量，长文先规划拆分。
- **改完大臣/举荐源表必须重建派生表**：ministers / recommend_ministers 的 code_name 变化后 minister_assets（与 recommend）不会自动更新——别等 validate 报 derived-diverged 才想起；derive-tables 幂等，改完主动跑。
- **新增实体缺资源即 asset-missing error**：大臣立绘四件套（chat/face/card/argue）、事件图、兵种通用图按约定请求键放 client-assets——原版已有实体回退官方图不报，新增实体没资源就是 error；按 validate 报错信息里的请求键逐一补齐。
- **改名后派生表残留旧行**：rename 落点清单含 minister_assets 等派生表的旧派生行——派生重建会保留额外段，旧行不自动消失；按落点清单删除旧行再跑 derive-tables 重建，否则旧实体名继续触发 asset-missing。

## 检查点

- [ ] 新增行的 code_name 不与任何表冲突（`validate` 会报 event-key-duplicate 等）
- [ ] 派生表已重建（大臣/举荐改动后无 derived-diverged）
- [ ] 删除/改名前已查 references 与 rename 落点清单，无新增悬空引用
- [ ] 风格与该 mod 既有数据一致（措辞、数值量级）
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** the user asks to add content to an EXISTING mod — new tables, entities (ministers/events/troops), rows, or to rename/delete existing entities. Building from scratch belongs to create-mod-from-scratch; pure numeric tuning belongs to balance-tuning.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Survey first: `list_tables` (existing vs missing tables), read existing rows to match the author's naming and magnitude style.
2. Check the contract: `schema_table` for fields/FKs; `vanilla_table` for official reference magnitude.
3. Incremental writes: append rows at the end of the data region, keep the 4-row metadata block intact; small batches with `validate` after each. Create missing tables via `mod_scaffold` `tables` (registers manifest) — never hand-create CSV files.
4. Derived tables & references: after ministers/recommend source changes run `derive_tables`; before rename/delete run `references` and `rename` (`dryRun: true`) — fix referencing cells first, then the definition row. New entities without conventional assets raise asset-missing errors — put files under `client-assets` at the exact reported request key.
5. Close out: `quality_gate` (report inherited vs new warnings separately), `preflight` must be valid; update `baseline_set` when world facts change.

**Top pitfalls:** always re-check `contractCoverage.missing` up front (writing to a ghost table fills nothing); registered texts have a 20,000 code-point hard cap (warning at 18,000); renamed entities leave stale rows in derived tables — delete them and re-run `derive_tables`.
