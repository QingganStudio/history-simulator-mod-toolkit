---
name: edit-mod-config
description: "修改《历史模拟器：崇祯》Mod 的注册 JSON 配置（历法 game_config、官职 minister_positions、根级 global_config 等）与 client-assets/field-definitions.json 字段显示覆盖。用户要改游戏配置、注册 JSON、字段中文标签/说明时使用；普通 CSV 表的增删行与调参走 extend-tables（扩表）或 balance-tuning（平衡调整）。 English triggers: edit game config JSON, registered JSON, field labels, calendar game_config, minister positions, field definitions override."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：编辑注册配置与字段覆盖

触发：用户要求修改游戏配置 JSON（历法、官职体系、视图配置等注册 JSON）、调整字段的中文标签/说明。

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。

## 先分清两类写入

| 形态 | 文件 | 语义 |
|---|---|---|
| 注册 JSON 全量替换 | manifest 登记的 JSON（如 `global/game_config.json`、`global/minister_positions.json`、根级 `global_config.json`、`pg-table-mapping.json`） | 整份覆盖官方同名配置——先查原版结构再整文件写全 |
| 字段显示稀疏覆盖 | `client-assets/field-definitions.json`（不登记 manifest） | 只写与官方基线不同的 label/description；未知 schema/字段/属性运行时被忽略并报 validate warning |

## 步骤

1. **查逻辑名与结构**：调用 MCP `schema_list` 看注册 JSON 逻辑名清单（kind=json）；用 `schema_table` 传逻辑名查字段树；用 `vanilla_json` 取原版内容作结构底本（巨型 GeoJSON 返回摘要）。
2. **全量替换**：按原版结构整文件写目标 JSON——键名、层级、类型以 `schema_table` / `vanilla_json` 为准，不发明新字段。注意两对双副本：`game_config`（`client-assets/game-config.json` ↔ `global/game_config.json`）与 `global_config`（根级 `global_config.json` ↔ `client-assets/global_config.json`）同存时必须逐字节一致（编辑器保存时双写；不一致 validate 报 `mirror-diverged`，以内容为准的一份覆盖另一份）。注册 JSON 里的实体引用键（如 `global_config.main_regime_code` → regimes.code_name）改动前先确认目标存在——跨表不一致 validate 报 `json-fk-target-missing` error。新文件落在注册命名空间（pg/、mongo/、global/、prompt/、text/、根级注册 JSON）必须登记进 manifest 的 files 节，否则游戏不加载（validate 报 `file-unregistered`）。
3. **稀疏字段覆盖**：先调用 `field_definitions` 查官方基线的有效 schema 与字段名；`client-assets/field-definitions.json` 只写改动项（`schemas.<名>.fields.<字段>.label/description`），与基线同值的项不写。未知键等于没写（validate 报 `unknown-schema`/`unknown-field`/`unknown-property` warning，运行时被忽略）。
4. **复验**：调用 `validate`（注册 JSON 结构、镜像一致、登记关系、字段覆盖对账）与 `preflight`；缺表与 `quality_gate` 结论分开报告。

## 高危踩坑

- **镜像副本漏写一边**：`game_config` / `global_config` 两对双副本只改一份即 mirror-diverged——以内容为准的一份覆盖另一份，保持逐字节一致。
- **引用键指向不存在的实体**：`global_config.main_regime_code` 等引用键必须指向既有表行——改名政权后忘记同步配置，validate 报 `json-fk-target-missing`；改引用键前先查 `references` 或目标表。
- **字段覆盖发明新键**：覆盖文件里的 schema/字段/属性名必须存在于官方基线（先查 `field_definitions`）；未知键运行时被静默忽略，等于没改。
- **注册 JSON 当增量补丁写**：注册 JSON 是全量替换语义——漏写的键不是"保留官方值"，而是彻底丢失；整文件照原版结构写全。

## 检查点

- [ ] 字段名与结构来自 `schema_table` / `vanilla_json` / `field_definitions`，未发明未知字段
- [ ] 双副本逐字节一致（无 mirror-diverged）；新文件已登记 manifest（无 file-unregistered）
- [ ] 稀疏覆盖只含与基线不同的项；validate 无 field-definitions 族 warning
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** editing registered game-config JSON (calendar game_config, minister_positions, root global_config, pg-table-mapping…) and the client-assets/field-definitions.json display override. Plain CSV row edits/number tuning belong to extend-tables / balance-tuning.

**Two write shapes:** registered JSON = full replacement (mirror vanilla structure, whole-file write); field-definitions.json = sparse override (only changed label/description keys — unknown keys are silently ignored at runtime and warned by validate).

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. `schema_list` (kind=json) → `schema_table` by logical name → `vanilla_json` for the official content as the structural template.
2. Write the whole file following vanilla structure — no invented fields. Two mirror pairs (`game_config`, `global_config` dual copies) must stay byte-identical (mirror-diverged otherwise). Reference keys (e.g. `global_config.main_regime_code` → regimes.code_name) must target existing rows (json-fk-target-missing otherwise). New files in registered namespaces must be added to the manifest files section (file-unregistered otherwise).
3. Sparse override: query `field_definitions` for valid schema/field names first; write only differing `schemas.<name>.fields.<field>.label/description` entries.
4. Re-verify with `validate` + `preflight`; report missing tables and `quality_gate` conclusions separately.

**Top pitfalls:** registered JSON is full-replacement semantics — keys you omit are LOST, not "kept from official"; unknown override keys equal "not written".
