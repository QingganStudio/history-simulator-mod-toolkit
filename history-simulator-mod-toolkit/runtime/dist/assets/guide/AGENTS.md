# AGENTS.md — 崇祯 Mod 制作工作流契约

> 本文件是给通用编码 agent（Claude Code / Codex / ZCode 等）看的工作流约定。
> 所有能力来自 `mod-toolkit`（MCP 工具或同名 CLI）。带 MCP 时优先用工具；否则用 `npx mod-toolkit <命令>`。
> 本文统一用 CLI 连字符形态书写工具名（如 `mod-scaffold`）；MCP 工具为同名下划线形态（`mod_scaffold`），两者互为等价别名，CLI 侧两种写法都接受。

## 角色与目标

你（agent）是 mod 协作者。用户会用自然语言提出制作需求（从零建 mod / 扩表 / 修错 / 平衡调整）。
**质量标准：`validate` 0 error + `preflight` valid + `quality-gate` 未裁定 issue 为 0 且已裁定无 error + 缺表 0（`contractCoverage.missing` 为空，无 `csv-import` 的 missing-table / table-misplaced）。** 质量来自"改动 → 校验 → 修复"的循环密度，不来自一次性生成。`quality-gate` 同时可能报 warning 和 error（如真实统治者重复的 `duplicate-ruler`）。有意设计的 warning 经用户确认后，可把指纹写进 `.mod-agent/quality-gate-ack.json`（数组项 `{table, rule, rowKey, column, reason}`——照报告里该 issue 的字段抄）。当前 ack 按指纹分组，不按 severity 限制；所以交付时必须检查 `acknowledged[]` 中也没有 error，不能只看 `valid`。指纹对不上时旧裁定会自动失效。

## 铁律（每次任务都遵守）

1. **写表前先查规范**：对要动的表先调 `schema-table <表名>`——字段类型、required、min/max、enum_values、外键目标都在里面；列的中文语义（display_name/description）也在这里。
2. **拿不准就查文档**：`docs-search <关键词>` → `docs-page <路径>`。游戏机制、命名约定、限制都有文档。
3. **数值要有参照**：写任何表前用 `vanilla-table <表名>` 看原版同类数值的量级（调平衡时逐项对照）。**完整 mod 的唯一权威参照是捆绑 vanilla（chongzhen-1627，27 表 + 全部注册 JSON + prompt 层全齐）**——做「改良原版」型 mod 用 `mod-scaffold <空目录> --from-vanilla [--id/--name/--author]` 一步 fork 完整副本（与编辑器用户等价起点，产物开箱双绿），然后按维度改；查单个 JSON 的正确写法用 `vanilla-json <逻辑名>`（巨型 GeoJSON 返回摘要）。
4. **每次改动后必须 validate**：用你的文件编辑工具直接改 mod 目录里的 CSV/JSON（linter 模式），改完立刻 `validate <目录>`，按错误码逐个修复，循环到 0 error。
5. **删除/重构前必查 references**：`references <目录> <表名> <code_name>`（`--direction out` 查该行自身引用了谁）——悬空引用是最高频的 mod 崩坏原因；**改 code_name 前先跑 `rename <目录> <表> <旧名> <新名> --dry-run`** 拿到按文件分组的全部落点清单（FK 单元格 / 事件 effect·condition / 注册 JSON / 地图省份要素 / asset 文件名）再逐一处理。
6. **尊重结构**：CSV 前 4 行是元数据块（含列名行），不要破坏；多行引号单元格不要拆物理行；`rows_locked` 的表不生成新行；`__` 开头的内部列不要动。
7. **`.mod-agent/scenario-baseline.json`** 只通过 `baseline-get` / `baseline-set` 读写（有 schema 校验）。
8. **新建剧本后必查 setting 层**：必须检查 prompt/setting/ 是否为本剧本内容——若为空或仍是原版，AI 将按原版世界观推演（用 `mod-scaffold --prompt` 生成骨架后逐文件填写）。
9. **prompt 层字段口径与数据层不同**：写 db_info 或其他 prompt 层文本（setting/example）引用外键字段时，统一写 `_code_name` 别名——`regime_id→regime_code_name`、`location_province_id→location_province_code_name`、`party_id→party_code_name`，别名与服务端工具契约参数同名，可提高推演 function call 命中率；`schema-table` 返回的 `_id` 列名只用于 CSV 表头与事件条件/效果 JSON 的 `column_name`，不要抄进 prompt 层（校验不报，属静默劣化）。
10. **新建行的 code_name 按编号段续编（官方推荐，不强制）**：段位表新行用 `{段前缀}{3 位编号}_{语义后缀}` 形态、从现有最大编号 +1 续编——省份 `1xxx_`、政权 `2xxx_`、大臣/举荐大臣 `3xxx_`（两表共享编号空间）、军队 `4xxx_`（陆海统一，原版海军 `Axxx_` 是历史遗留）、兵种 `5xxx_`、装备 `6xxx_`、阶层 `7xxx_`、土地类型 `8xxx_`、船 `Bxxx_`、党派 `Cxxx_`，形如 `1001_beijing`、`B001_fu`；后缀用小写拼音/英文。这不是校验要求（validate/preflight 不拦），但编号段能帮助推演模型按编号开头快速定位条目所在表。完整编号段表与续编习惯见 `docs-page mod-maker/data-rules.md`。

## 校验错误怎么办

- `validate` 报告的绝大多数 issue 带 `reasonCode`（稳定）+ `message`（中文），修复时以 reasonCode 为准（例外：finances 单例表多行、province_land_holdings 的 code_name 派生不一致这两个结构校验不带 reasonCode，直接按 message 处理）：
  `primary-key-missing/duplicate`、`foreign-key-missing`、`invalid-enum`、`below-minimum/above-maximum`、
  `event-key-duplicate`、`perk-relation-invalid`、`fixed-turn-event-invalid`、`super-event-snapshot-invalid` 等。
- warning 级（导入修复类）可以容忍但要知道原因；error 级必须清零。
- 跨文件引用完整性（`asset-reference` 码）：`mirror-diverged`（注册 JSON 双副本内容分歧，以一份覆盖另一份）、`file-unregistered`（注册命名空间文件未登记 manifest，不会被游戏加载——表 CSV 用 `mod-scaffold --table` 补登记；prompt 分层键、文本、根级 JSON 等 `--table` 覆盖不到的，直接手改 manifest.yaml 的 `files` 节登记）、`asset-missing`（新增实体的约定资源缺失：事件/兵种/技能图标/省份浮层/立绘四件套——放文件进 client-assets 或显式映射；原版已有实体回退不报）。资源请求键约定（省份浮层/兵种图变体回退链/事件图分目录/立绘四件套键形）的唯一权威是 `docs-page resources-and-ui/images-and-assets/registered-images.md`（插件形态另有 manage-mod-assets 技能带全流程）；覆盖放好后用 `asset-map` 确认请求键真实命中 mod 文件（索引未命中会静默回退官方资源，页面正常显示不算数）。孤儿政权、孤儿兵种、无法识别、未映射格式和被遮蔽文件由 `validate` 报 error；不进入上传 `preflight`。
- 派生表一致性（`derived-table` 码）：`derived-file-missing`（大臣两表有 code_name 但 minister_assets.csv 缺失）、`derived-diverged`（文件与两表现算派生不一致）——都跑 `derive-tables <目录>` 重建修复；改完 ministers / recommend_ministers 的 code_name 应主动重跑。
- 字段元数据覆盖对账（`field-definitions` 码，validate-only warning，不拦上传）：`unknown-schema` / `unknown-field` / `unknown-property`（client-assets/field-definitions.json 覆盖里的 Schema / 字段 / 属性键不在官方基线——运行时只忽略+打诊断，写了等于没写；写覆盖前先用 `field-definitions` 工具查基线的有效名）、`invalid-shape`（覆盖文件 root/schemas/fields 非对象）。稀疏覆盖合法，不报覆盖不全。
- 导入层诊断（`csv-import` 码，validate-only warning，不拦上传）：`missing-table`（表 CSV 缺失、游戏按空表处理——用 `mod-scaffold --table` 补建并登记）、`table-misplaced`（表放错 group 目录，如 ministers.csv 放进 global/——移到 `expected` 参数指向的归属路径）、`metadata-block` / `duplicate-header` / `empty-header` / `required-column-missing` / `column-default-filled` / `ragged-record` / `cell-invalid`（元数据块、表头、缺列、列数不齐、单元格类型——按 issue 的表/行/列定位修复）。validate 报告的 `contractCoverage.missing` 应为空——缺表 0 才算数据面齐备。
- prompt 层巡检（`prompt-layer` 码，validate-only warning，不拦上传）：`setting-missing`（声明 ScenarioMeta 的剧本 mod 缺 setting 层文件，聚合单条，`missing` 参数列全缺失名单——用 `mod-scaffold --prompt` 生成 10 个骨架并分层登记；纯数据 mod 不报）、`setting-empty`（setting 文件存在但空白——填写本剧本内容或删除）。**setting 层缺失/空白时 AI 将按原版（崇祯）世界观推演**——新建剧本后必须检查 prompt/setting/ 10 个文件是否已写成本剧本内容。字符余量预警（`text-char-limit/approaching-limit`）：注册的 md/txt/yaml 接近硬顶即预警（具体阈值数字的唯一权威是 `codes --code text-char-limit`，预警线 = 硬顶 90%），收到就应精简，别等撞 upload 墙。
- 结构校验之外再跑 `quality-gate`：财政失衡、驻地矛盾、民心-叛乱联动异常等设计层启发式通常为 warning；同一真实 `current_ruler` 重复出现在多个政权时报 `duplicate-ruler` error，应修正身份。“纷争内斗”是无单一统治者的精确占位值，不参与人物判重。未裁定 warning 要修正或说明设计意图。

- 地图覆盖：1.6.0 官方底图已含旧地名；沿用底图时，`validate` 的 `legacy-map-text-overlap` warning 提醒移除重叠旧文字，自定义新 `text` 合法。制作地图时查 `docs-page resources-and-ui/map-and-provinces/geojson-contract.md`。
- 历法配置：春为岁首，冬转春才翻年；显式回合文案优先。改开局季节或年号时查 `docs-page advanced/game-and-ui-config/calendar-config.md` 并在编辑器预览逐回合文字。

## 发布前必过 preflight

`validate` 覆盖 CSV 数据面、注册 JSON 结构校验（**内环即报**：目录内存在的注册 JSON——历法/视图/官职体系/GeoJSON 等——与 preflight 同码同文，缺文件静默因 JSON 全部是可选覆盖）、ScenarioMeta schema 校验（**内环即报** error：根形状 / 13 字段严格类型 / seasons 恰 4，与 preflight 同码同文）、JSON 跨表一致性（`main_regime_code`↔regimes、地图省份要素↔provinces 为 error；省份不上图/势力配色/官职体系职位名唯一为 warning）、跨文件引用（asset-reference）、派生表一致性、字段覆盖对账、导入层诊断、prompt 层巡检（见上方分节，均不拦上传）；`preflight <目录>` 是游戏上传 Workshop 的**整目录门禁**，与本工具输出逐码一致：manifest 严格校验、注册文件、Config/assets.json 显式映射、client-assets 扩展名/大小/像素预算、立绘表额外行、prompt 合成变量、custom.css 1MB 上限。**宣布 mod 完成前必须 `preflight` valid。** 每条 issue 带 `severity`，但上传门禁零容忍——warning 级（如 `text-char-limit`、`prompt-variable-undeclared`）同样拦截，要修到 issues 清空而不是只清 error。

## 禁止事项

- 不要发明字段名/表名——一切以 `schema-table` 返回为准（唯一例外：prompt 层文本引用外键字段时按铁律 9 的字段口径写 `_code_name` 别名，不用 `_id` 列名）。
- 不要绕过校验直接宣布"完成"——完成 = 上方质量标准四件套（validate 0 error + preflight valid + quality-gate 未裁定 issue 为 0 且已裁定无 error + 缺表 0）。
- 不要修改游戏本体的任何文件；mod 目录之外只读。
- 不要把 API key 或用户隐私写进 mod 数据。

## 任务类型

见 `skills/` 目录：从零建 mod（create-mod-from-scratch）、扩表（extend-tables）、事件与决议创作（create-events）、国策树创作（build-perk-tree）、地图制作（custom-map）、修错（repair-errors）、平衡调整（balance-tuning）、prompt 层改写（rewrite-prompt-layers）。按任务类型先读对应 skill 再动手。插件形态（Agent Plugin 分发）另有两份插件专属技能 edit-mod-config / manage-mod-assets，由宿主技能面自动发现，不经本文件分发。
