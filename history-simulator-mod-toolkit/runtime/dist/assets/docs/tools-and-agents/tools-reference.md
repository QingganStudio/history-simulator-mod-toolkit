---
title: 工具参考
description: 22 个 MCP 工具与 23 个 CLI 命令的对照表
---

# 工具参考

> 适用范围：mod-toolkit 内核注册面（MCP 22 个工具 / CLI 23 个命令，含 1 个 CLI 专属命令）；最近代码核对：2026-09-25；验证层级：双绑定同一内核，全部命令输出 JSON。

## 总则

MCP 工具名用下划线（如 `schema_table`），CLI 命令名以连字符为规范形态（如 `schema-table`），下划线为等价别名（`mod_scaffold` ≡ `mod-scaffold`）——两者同名、参数大体对等，差异已在各工具描述写明（如 `baseline_set` 的 MCP 传对象、CLI 传 json 文件）。下表覆盖全部 22 个 MCP 工具与 CLI 专属的 `vanilla-list`，共 23 个 CLI 命令。除帮助与版本外全部输出结构化 JSON；MCP server 同时以 `guide://` 资源暴露工作流契约。文档、原版数据与官方字段基线的查询根目录默认随包分发，可用环境变量 `MOD_TOOLKIT_DOCS_DIR` / `MOD_TOOLKIT_VANILLA_DIR` / `MOD_TOOLKIT_FIELD_DEFINITIONS_DIR` 覆盖。

## 校验与门禁

| MCP / CLI | 输入 | 输出与退出码 | 适用场景 |
|---|---|---|---|
| `validate` | Mod 目录（可选 `limit` / `offset` 窗口参数） | 结构化报告：`schemaVersion`、`valid`、`errorCount` / `warningCount`、逐条 issue（表 / 行 / 列 / reasonCode / message）、`contractCoverage`（27 张表的 `presentTables` / `missing` / `misplaced` 落实清单 + 剧本 mod 缺失的 setting 层 `missingSettingPrompts` + prompt 槽位视图 `promptLayers`（14 条目 × 层：`source=mod/absent/no-slot`、newSlot 开新槽标记、字符预算 budget、不可解析变量）与惰性名 `inertPromptNames`）、`manifestMissing`、`tablesFound`；不合法退出码 1 | 覆盖主键、字段类型 / 枚举 / 范围、跨表外键、事件 key 跨表唯一、国策关系、固定回合事件、超事件快照、单例表、结构化引用、导入层诊断（缺表 / 表放错 group 目录 / 元数据块 / 表头 / 缺列 / 列数不齐 / 单元格类型，`csv-import` 码 warning 级）、跨文件引用完整性（双副本镜像一致、注册命名空间文件登记、新增实体资源存在性）与 prompt 层巡检（setting 层缺失 / 空白 `prompt-layer` 码、文本超 18,000 码点的 `text-char-limit/approaching-limit` 余量预警，均 warning 级不拦截上传）。**每轮改动后必跑** |
| `preflight` | Mod 目录 | 发布门禁整目录预检：`contractVersion` 2、`valid`、逐条 issue（含 severity 与 `rule` 修复指引文案——码标题 + reason 指引合成，输出层注入；超 100 条截断标记）；不合法退出码 1 | manifest 严格校验、注册文件、CSV 行列与单元格限制、JSON / YAML 语法、`Config/assets.json` 显式映射、client-assets 扩展名 / 大小 / 像素预算、立绘表额外行、prompt 合成变量、custom.css 1MB 上限。**宣布完成前必过**，语义见[发布门禁与 preflight](./preflight-and-publish) |
| `quality_gate` | Mod 目录（MCP 传 `tables` 表名数组、CLI 用 `--table`，限定检查范围；另可选 `limit` / `offset` 窗口参数） | 语义启发式 warning/error 清单（同一真实 current_ruler 重复的 duplicate-ruler 为 error；“纷争内斗”精确占位值豁免）：报告外壳含 `valid`（未裁定 `issueCount === 0`）、`issueCount`（未裁定数）、`acknowledged`（已裁定节——`.mod-agent/quality-gate-ack.json` 指纹 `{table, rule, rowKey, column, reason}` 命中的 issue；交付只认可有意 warning）、`ackFile`、逐条 issue（`rule` 规则名与 `reasonCode` 别名双字段并存）；数值建议范围（`numeric-range`，边界读 schema 契约 `advisoryMin`/`advisoryMax`，分档 0-100 / 0-10 / 0-1，如民心 public_sentiment、城防 defence_level、饷率 salary_rate）、必填文本列非空（`text-nonempty`，列集取 schema `required` 声明的 text/varchar 列，single source of truth）、单回合收支失衡、驻地与政权矛盾、民心-叛乱联动异常、零存在感政权死条件（`domain-dead-regime-condition`：事件条件断言某郡归某政权，而该政权零郡且零军队 → 条件永不满足、事件永不触发）、零存在感政权死效果（`domain-dead-regime-effect`：事件效果引用零郡零军队且未被任何效果赋予郡/军队的政权 → 效果等于没写；被效果赋予郡/军队的「将崛起」设计不报）、同回合同条件事件（`domain-same-turn-events`：fixed_turn_events 同回合剥掉 turn 后条件完全相同的 ≥2 条事件，同回合齐发编排事故）、数值偏离场景基线等；合法叛军/海权设计（零郡但有军队）在两条零存在感规则下均不报；必填文本空值与 validate 的 error 级 `required-value-missing` 是**有意双保险**——validate 拦结构、quality-gate 做体检；issue 另带 `category`（convention = 命名约定建议 / quality = 语义检查，D21 分级） | error 清零后的设计层检查；未裁定 warning 要修正或说明设计意图；error 须修正数据，不作为有意设计豁免（convention 类有意偏离可直接 ack 豁免） |

ack 实现按 `{table, rule, rowKey, column}` 指纹归档，当前不限制 severity；因此 `quality_gate.valid` 只表示未裁定区为空。交付时还须检查 `acknowledged[]` 无 error，真实统治者重复等 error 应修正。

## code_name 前缀约定表（convention 级）

`code-name-prefix` 是 convention 级建议（源自原版命名习惯；其他朝代 mod 可有意偏离——偏离后写入 `.mod-agent/quality-gate-ack.json` 豁免即可，不再是每次重裁的噪音）。完整约定（与内核前缀表同源）：

| 表 | 前缀 | 形态 | 示例 | 原版编号段（参考量级） |
|---|---|---|---|---|
| `provinces` | `1` | 1 + 3 位数字 + `_` | `1001_beijing` | 1001-1030 |
| `regimes` | `2` | 2 + 3 位数字 + `_` | `2001_ming` | 2001-2017 |
| `ministers` | `3` | 3 + 3 位数字 + `_` | `3001_guanyu` | 3001-3101 |
| `recommend_ministers` | `3` | 与 ministers 同段（同一人） | `3001_guanyu` | 随 ministers |
| `parties` | `C` | C + 3 位数字 + `_` | `C001_jingzhou` | C001-C006 |

新表行编号建议接着原版段往后排（如政权 2018 起），避免与原版实体撞 code_name；mod 内自定义编号体系时保持「前缀 + 数字 + `_` + 英文标识」形态即可过约定。

## 三工具 issue 字段对照表

`validate` / `preflight` / `quality_gate` 三份报告的逐条 issue 字段并不完全同构，脚本化消费按下表解析（✓* = 可选字段，不出现时不带键）：

| 字段 | `validate` | `preflight` | `quality_gate` | 含义 |
|---|---|---|---|---|
| `code` | ✓* | ✓ | — | 错误大类码（47 码目录，用 `codes` 查询） |
| `reasonCode` | ✓* | ✓* | ✓ | 稳定具体原因，修复时以此为准；quality_gate 侧是 `rule` 的别名（同值） |
| `rule` | — | ✓* | ✓ | **两工具语义不同**：preflight 侧是修复指引文案（码标题 + reason 指引合成，输出层注入，未知码省略）；quality_gate 侧是规则名（如 `domain-troop-location-regime`） |
| `severity` | ✓ | ✓ | ✓ | `error` / `warning` |
| `message` | ✓ | ✓ | ✓ | 中文具体消息（含定位细节） |
| `table` | ✓ | ✓* | ✓ | 涉及的表名 |
| `column` | ✓* | ✓* | ✓* | 涉及的列名 |
| `rowKey` | ✓* | — | ✓* | 行定位（validate 用 `__rowId`；preflight 用下行两个字段代替） |
| `dataRow` / `physicalLine` | — | ✓* | — | CSV 数据行号 / 物理行号 |
| `relativePath` / `logicalName` | — | ✓ / ✓* | — | 文件相对路径 / 注册 JSON 逻辑名 |
| `id` / `level` | ✓ / ✓ | — | — | 去重 ID / 校验级别（field / row / cross_table） |
| `reasonData` | ✓* | ✓* | — | 受控短参数（键集合见 `codes --code` 的 params） |
| `actual` / `limit` | — | ✓* | — | 超限类 issue 的实际值 / 限制值 |

报告外壳的通过判据统一为 `valid`：validate 是 `errorCount === 0`，preflight 是门禁全过（warning 也拦），quality_gate 是未裁定 `issueCount === 0`（ack 豁免的有意设计不计入）。

## 查询与参照

| MCP / CLI | 输入 | 输出与退出码 | 适用场景 |
|---|---|---|---|
| `schema_list` | — | 全部数据表与注册 JSON 逻辑名清单（`kind` 区分 csv/json；表含是否附列中文语义，JSON 含物理路径） | 了解表与注册 JSON 的全貌与分层 |
| `schema_table` | 表名或注册 JSON 逻辑名 | 表：字段定义、类型、枚举、min/max、外键目标与列中文语义；JSON：字段树、类别与物理路径（按 `kind` 判别返回形状）；不在 schema 中退出码 2 | **写任何表或注册 JSON 之前必查**；拿不准字段名时以它为准 |
| `docs_search` | 关键词（大小写不敏感，不能为空或纯空白） | 命中的文档文件、行号与摘录；单次最多 20 条，`totalCount` / `hasMore` 明示余量，`offset` 续读后面的命中 | 查游戏机制、命名约定与限制的官方说明 |
| `docs_page` | 文档相对路径 | 单篇文档全文 | 深读某一篇（本站的全部页面均可检索） |
| `vanilla_table` | 表名，可选按 `code_name` 过滤到单行 | 原版数据行（schema 感知反序列化） | 数值量级参照与措辞风格样本 |
| `vanilla_json` | JSON logical name（15 个注册名之一），可选根目录覆盖 | 原版注册 JSON 内容（4 个 manifest JSON + 11 个 client-assets JSON；`game_config` / `global_config` 双注册名两形态内容相同）；超 64KiB 时返回有界结构摘要：地图列要素、字段定义列 schema 与字段数、其他 JSON 列顶层结构；key 非法时列出全部有效 key | 全量替换语义下照原版结构写注册 JSON；字段详情用 `field_definitions` 带 schema 名查 |
| `vanilla_prompt` / `vanilla-prompt` | 逻辑名（14 个）、可选 `layer`（setting/rules/example）、`dir?` | 指定层的原版原文 + 元信息：availableLayers（官方层组合）、renderKind（jinja/static/yaml）、runtimeWeight（fatal/fallback）与 runtimeNote、variables（仅廷议 11 个）、hint（坑位提示）、content（>64KiB 截断标 truncated） | 重写 setting/example 层前取官方写法参照；改 prompt 前先读 hint（缓存键/字段别名同步等坑） |
| `field_definitions` | 可选 schema 名，可选根目录覆盖 | 无参：官方字段基线全部 51 个 schema 清单（名字 + 字段数 + 命名类别 `api-response` / `table-named`）；带 schema 名：该 schema 字段树（name / label / description / type / ref），未知名退出码 2 并列出全部有效名 | 写 `client-assets/field-definitions.json` 覆盖前必查——schema 名与字段名以官方基线为准。当前剧本镜像的 `vanilla_json field_definitions` 含 51 个 schema / 455 字段，与游戏 public 官方基线语义一致；该大型 JSON 返回 schema 摘要，逐字段详情用本工具带 schema 名查询 |
| `codes` | 可选 `--code` 错误码、可选 `--severity` 过滤（MCP 同参 `code` / `severity`） | 无参：47 个内容预检错误码清单（code + severity + title + reason 数）；`--code`：该码全部 reason 与修复指引文案（reasonCode / title / rule / params），未知码退出码 2 并列出全部有效码；`--severity`：按码级 severity 过滤清单（非法值退出码 2） | 按 code / reasonCode 修 issue 前查指引，不靠 grep 源码；validate / preflight 报错现场的第一站 |
| `asset_map` | Mod 目录，可选 `limit` / `offset` 开窗（缺省窗口 200，`totalCount` 恒全量） | 资源覆盖终态清单：目录约定与 `Config/assets.json` 显式映射合并后的「请求键 → 文件」清单（`via` 区分 convention / explicit）与被拒清单 `rejected`（含原因码）；`truncated` / `hasMore` / `nextOffset` 显式 | 替换资源后必须用它确认请求键真实命中——资源索引未命中会静默回退官方资源，「页面能显示」不算覆盖成功 |
| `vanilla-list`（仅 CLI） | — | 原版全部表的行数清单 | 定位「原版哪张表有多少数据」 |
| `list_tables` / `list-tables` | Mod 目录 | 逐表行数与导入告警（缺失表按空表处理，`validate` 报告 `contractCoverage` 的目录级对照面） | 接手现有 Mod 时先看数据现状 |

## 影响面、脚手架与项目文件

| MCP / CLI | 输入 | 输出与退出码 | 适用场景 |
|---|---|---|---|
| `references` | Mod 目录 + 表名 + code_name + 可选 `direction`（`in` / `out`，缺省 `in`，MCP/CLI 双形态同参 `--direction`；另可选 `limit` / `offset`） | `in`：该实体被哪些表、哪些行引用（含 map 列展开）；`out`：该行自身引用了哪些表的哪些行（FK 列值 + 事件 effect/condition 内 `row_name` 与经 FK 列的 `value` + `decisions.super_event` 快照），报告带 `direction` 字段 | **删除、改名、重构前必查**，防悬空引用；改名前用 `out` 掌握全部传播点 |
| `rename` | Mod 目录 + 表名 + 旧 code_name + 新 code_name + `--dry-run`（CLI；MCP 形态 dryRun 参数必须为 true——本期只读不写盘） | 改一个 code_name 的全部落点按文件分组（`files[].locations` + `totalCount`，`written: false`）：定义行 / schema FK 引用列 CSV 单元格（含 map 键与数组）/ perks 技能引用列 / 事件 effect·condition 内 `row_name` 与经 FK 列的 `value` / `decisions.super_event` 快照 / 注册 JSON 键值（`global_config.main_regime_code`、省份视图 `colors` 键、地图省份要素 `code_name`——provinces 改名联动 GeoJSON）/ 派生立绘表行 / asset 各族约定文件名（事件图含 random 点换连字符换算与 legacy 点号变体、兵种通用图与势力变体（含有效显式映射）/ 技能 lock·unlock / 立绘四件套 / 省份浮层 `dynamic-assets/province-overlays/<code_name>.webp` / 政权驻军图标）与 `Config/assets.json` 显式映射；旧名不存在返回空清单与提示（不报错）；无 `--dry-run`、未知表、单例表、新旧同名退出码 2 | 最高频重构不再靠 grep 手改 N 个文件——改名前先拿落点清单逐一处理 |
| `scaffold` | 表名 | 该表的依赖骨架默认行（JSON 输出，不直接写盘，由 agent 抄进 CSV） | 新建行时起步 |
| `mod_scaffold` | Mod 目录 + 可选身份参数、表名列表、`prompt` 开关（可传 A,B 名单只开部分槽）、`example` 开关（附带生成 6 个默认 example 槽骨架）、`dryRun` 开关，或 `fromVanilla` 开关（CLI `--from-vanilla`，与表名/`prompt` 互斥、目标目录须为空） | 生成整 Mod 毛坯并回读校验：`manifest.yaml`（files 只登记真实生成文件，经**保注释最小编辑**写入——手写注释 / 锚点 / 键序存活）、`Config/assets.json`、目录树、每张选定表的 CSV（4 行元数据块 + 骨架默认行，元数据块与编辑器产物逐字节一致）；`prompt` 开关生成 `prompt/ScenarioMeta.yaml` 骨架 + setting 层 10 个剧本专属骨架（名字与原版镜像逐字对齐，CourtDiscussionPrompt 自带 11 个必填变量占位），按 `<Name>.setting` 分层键登记（required/layer 叶；rules 层从捆绑 vanilla 逐字节拷贝并登记；example 层可选生成）；对已有目录仅支持增量加表（建缺失 CSV + 补登记），不覆盖已有 CSV / 骨架 / rules；`dryRun` / `--dry-run` 只返回变更计划零落盘；派生表拒绝；`fromVanilla` 走 fork 通道——拷贝捆绑 vanilla（chongzhen-1627）全量 + manifest 身份四字段改写（files 登记原样），产物开箱即过 validate/preflight，「改良原版」型 mod 的完整起点（守卫对齐编辑器 fork：空目录、不得位于 vanilla 根内） | 从零创建或改良原版的第一步；生成物直接 `preflight` valid |
| `derive_tables` | Mod 目录，可选派生表名（`minister_assets` 或 `recommend`）、`dryRun` 开关 | `minister_assets` 从大臣两表的 code_name 现算派生集，并与既有文件额外行合并；`recommend` 从 `recommend_base` × `recommend_ministers` 按 code_name 匹配后全量重建，未匹配行跳过。两者都幂等，未登记时补 manifest；`dryRun` / `--dry-run` 只返回变更计划零落盘。报告计数按所选派生表分别给出 | 大臣两表改名后重建 minister_assets；举荐两表改动后用 `--table recommend` 重建 recommend |
| `baseline_get` | Mod 目录 | `.mod-agent/scenario-baseline.json` 的场景基线（世界观 / 势力 / 大臣状态） | 跨会话保持世界观一致 |
| `baseline_set` | Mod 目录 + 基线对象（MCP 传对象；CLI 传 JSON 文件路径） | 校验通过后写入；结构不符报错且不落盘 | 世界观事实变化时更新 |
| `guide_init` | 目标目录 | 把工作流契约（`AGENTS.md` 与同内容 `CLAUDE.md`）、八份任务 skills、两份 schema 快照、`.claude/skills/<name>/SKILL.md` 薄封装（frontmatter + 指向 skills 文件，进宿主 agent 技能注册表）写入目录；内容逐字节相同则跳过，覆盖前自动备份旧版到 `.mod-agent/guide-backup/<timestamp>/`（保持相对目录结构）；返回值逐文件标 `created` / `overwritten` / `unchanged`；打包副本与仓库源不一致时报错并指明重跑 build | 新 Mod 项目开工前执行一次 |

## 写入面清单

内核只有下表四个工具会写盘，其余（`validate` / `preflight` / 查询类 / `scaffold` / `references` / `rename` / `quality_gate` / `baseline_get`）对 Mod 目录只读：

| 工具 | 写入内容 | 写入方式 |
|---|---|---|
| `guide_init` | 20 个文件：`AGENTS.md` + 同内容 `CLAUDE.md` + 8 份任务 skills + 2 份 schema 快照 + `.claude/skills/<name>/SKILL.md` ×8 薄封装；覆盖时旧版备份到 `.mod-agent/guide-backup/<timestamp>/` | 内容逐字节相同跳过（`unchanged`）；覆盖前自动备份再写（`overwritten`）；新文件直接创建（`created`）；返回值逐文件标三态 |
| `mod_scaffold` | 选定表 CSV 骨架、`Config/assets.json`、目录树、可选 prompt 骨架（ScenarioMeta + setting 层 10 文件）；`manifest.yaml` 登记；`fromVanilla` 则 fork 捆绑 vanilla 全量 + 身份改写 | 已有 CSV / 骨架不覆盖只跳过；manifest 经保注释最小编辑登记（手写注释 / 锚点 / 键序存活） |
| `derive_tables` | `global/minister_assets.csv`；`manifest.yaml` 补登记 | 内容变化才重写（幂等）；manifest 经保注释最小编辑登记 |
| `baseline_set` | `.mod-agent/scenario-baseline.json` | 结构校验通过后整体写入；校验不符报错且不落盘 |

`mod_scaffold` 与 `derive_tables` 支持 `dryRun`（MCP）/ `--dry-run`（CLI）：只返回变更计划（`created` / `skipped` / `registered` / `manifestChanges`，`written: false`）零落盘，可先预览再动手；计划与随后真实执行的结果一致。

## 退出码约定

- **0**：成功。
- **1**：仅 `validate` / `preflight` 判定 mod 不合法（`valid: false`）——是 mod 内容的问题，报告里逐条 issue 按码修复即可（CSV 语法错等数据面问题也走这条路，不再表现为工具崩溃）。
- **2**：其余一切失败——参数缺失、未知命令 / 表名 / key、目录不存在、内部异常等工具自身或用法故障。

因此这两道校验可以直接作为 CI 流水线的门禁步骤使用，无需解析 JSON：「≠0 即失败」语义不变，1 与 2 还能进一步区分「mod 不合法」与「工具故障」。
