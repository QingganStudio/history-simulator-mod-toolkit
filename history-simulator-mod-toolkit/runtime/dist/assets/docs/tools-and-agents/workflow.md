---
title: Agent 工作流契约
description: 十条铁律、改动 → 校验 → 修复循环、任务 skills 导读
---

# Agent 工作流契约

> 适用范围：`mod-toolkit/guide` 工作流契约（AGENTS.md）与任务 skills；最近代码核对：2026-10-08；验证层级：契约文件随 mod-toolkit 分发，`guide_init` 写入 Mod 目录（内容相同跳过、覆盖前备份、逐文件报三态）。

## 契约是什么、怎么落盘

工作流契约是写给通用编码 agent 的一页约定（`mod-toolkit/guide/AGENTS.md`），定义 agent 与人协作做 Mod 时的角色、质量标准、铁律和禁止事项。用 `guide_init` 工具（CLI 同名命令）把它写进 Mod 目录后（同时落一份同内容的 `CLAUDE.md`、全部任务 skills 与 `.claude/skills/` 薄封装），支持读取工作区约定的 agent 进入该目录即自动遵守；不支持的平台也可以直接把契约文本贴进 agent 配置。

契约定义的统一质量标准：**`validate` 0 error + `preflight` valid + `quality-gate` 未裁定 issue 为 0 且已裁定无 error + 缺表 0**。质量门可含 warning 与 error，真实统治者重复的 `duplicate-ruler` 是 error；ack 当前按指纹归档，交付时还要检查 `acknowledged[]` 无 error。

## 十条铁律

1. **写表前先查规范**：对要动的表先查 `schema_table`——字段类型、必填、min/max、枚举、外键与列的中文语义都在里面；
2. **拿不准就查文档**：`docs_search` 检索关键词，`docs_page` 读全文——游戏机制、命名约定、限制都有官方文档；
3. **数值要有参照**：写任何表前用 `vanilla_table` 看原版同类数值的量级（调平衡时逐项对照）；完整 Mod 的唯一权威参照是捆绑 vanilla，做「改良原版」型 Mod 用 `mod_scaffold --from-vanilla` 一步 fork 完整副本；
4. **每次改动后必须 validate**：直接改文件（linter 模式），改完立刻校验，按错误码逐个修复，循环到 0 error；
5. **删除、重构前必查 references**：悬空引用是最高频的 Mod 崩坏原因；改 `code_name` 前先跑 `rename --dry-run` 拿到按文件分组的全部落点清单（FK 单元格、事件 effect·condition、注册 JSON、地图省份要素、asset 文件名）再逐一处理；
6. **尊重结构**：CSV 前 4 行元数据块不破坏、多行引号单元格不拆物理行、`rows_locked` 的表不生成新行、`__` 开头内部列不动；
7. **场景基线只经工具读写**：`.mod-agent/scenario-baseline.json` 只通过 `baseline_get` / `baseline_set` 访问（带 schema 校验）；
8. **新建剧本后必查 setting 层**：`prompt/setting/` 10 个文件必须写成本剧本内容——若为空或仍是原版，AI 将按原版世界观推演；
9. **prompt 层字段口径与数据层不同**：prompt 层文本（setting/example）引用外键字段统一写 `_code_name` 别名——`regime_id→regime_code_name`、`location_province_id→location_province_code_name`、`party_id→party_code_name`，别名与服务端工具契约参数同名；schema 的 `_id` 列名只用于 CSV 表头与事件条件/效果 JSON 的 `column_name`，不要抄进 prompt 层（校验不报，属静默劣化）；
10. **新行 code_name 按编号段续编（官方推荐，不强制）**：省份 `1xxx_`、政权 `2xxx_`、大臣/举荐大臣 `3xxx_`（共享编号空间）、军队 `4xxx_`（陆海统一）、兵种 `5xxx_`、装备 `6xxx_`、阶层 `7xxx_`、土地类型 `8xxx_`、船 `Bxxx_`、党派 `Cxxx_`，从现有最大编号 +1 续编，后缀用小写拼音/英文；完整编号段表与续编习惯见[数据规则](/mod-maker/data-rules)。

配套的禁止事项：不发明字段名与表名（一切以 `schema_table` 返回为准）；不绕过校验宣布「完成」；不改游戏本体文件（Mod 目录之外只读）；不把密钥或隐私写进 Mod 数据。

## 改动 → 校验 → 修复循环

这是整个工作流的发动机，也是[工具链总览](./overview)所说「质量来自循环密度」的落点：

1. agent（或你）按需求直接编辑目标文件——小批量，一次只动一张表或一组相关字段；
2. 立刻 `validate`，得到带 `reasonCode` 的结构化 issue 清单；
3. 按 reasonCode 分组逐类修复（`primary-key-duplicate` 去重、`foreign-key-missing` 补引用或补目标行、`invalid-enum` 对照 schema 改值……），而不是凭感觉重写；
4. 回到 1，直到 0 error；再跑 `quality-gate` 处理设计层警告，跑 `preflight` 确认可发布。

`validate` 的 warning 级（导入修复类）可以容忍但要知道原因；error 级必须清零。错误码的逐码修复指引见[错误排查](../reference/troubleshooting)。

## 任务 skills 导读

契约随附一组按任务类型组织的 skills（完整清单的单一事实源是契约 `AGENTS.md` 的「任务类型」节），按需求选用（`guide_init` 会一并写入目录）：

| 任务 | 流程要点 |
|---|---|
| **从零创建** | 对齐世界观（`baseline_set` 落盘）→ `mod_scaffold` 一次生成毛坯（manifest 登记 + 每表 CSV 元数据块与骨架行）→ 按依赖顺序填数据（每表先 `schema_table`，数值参照 `vanilla_table`）→ 大臣两表 code_name 定稿后跑 `derive_tables` 重建立绘派生表 → 逐表 validate → 收尾 `preflight` 确认 valid 后报告产出 |
| **扩表现有 Mod** | `list-tables` 看现状 → 读既有行理解命名与数值风格 → 小批量增量写入、每批 validate → 涉及世界观事实变化时更新基线 → 交付前 `preflight` |
| **事件与决议创作** | 按触发方式选型六类事件/决议表 → 定 `code_name` 先查重（随机三表共用命名空间）→ 写条件与效果（13 张可写表闭集、finances 目标省略行选择器）→ 逐批 validate 按事件专属码修 → 事件图放置后用 `asset_map` 核对命中 → `preflight` 收尾 |
| **国策树创作** | 四个正式分类选型 + `Skill.NNNN` 命名 → 主前置/完整前置体系与互斥双向维护 → 四类效果字段分工 + 两态图标 → 逐批 validate 修 `perk-relation-invalid` → `preflight` 收尾 |
| **地图制作** | 先做底图-text 策略决策 → 省份业务数据先行（地图编辑器不建业务记录）→ 按 GeoJSON 契约写五类要素（设计空间 1920×1080）→ `code_name` 全落点对账（改名先 `rename --dry-run`）→ validate 清零 `json-fk-target-missing` → `asset_map` 核对地图真实命中 |
| **修复错误** | 先跑 `validate`（或从被拒的 `preflight` 报告）拿全量清单，按 reasonCode 分组 → 逐类修复、删除类操作前必查 `references` → 修完回归四件套 → 按「错误码 → 修了什么 → 结果」汇报 |
| **平衡调整** | `schema_table` 定位字段 + `docs_search` 查机制说明 → `vanilla_table` 取参照、算变化幅度 → 一次只改一组联动字段并 validate → `quality-gate` 复核语义 → 按「表.列：旧值 → 新值（理由）」给用户确认 |
| **prompt 层改写** | 先查 `vanilla_prompt` 的层组合与渲染方式 → 按需求重写 setting/example 层，保留 rules 权威副本 → 运行 prompt 相关校验与 `preflight` → 报告改动与剩余限制 |

各 skill 的完整步骤与检查点在 `guide_init` 写出的 `skills/` 目录里，这里只做导读。逐个工具的输入输出见[工具参考](./tools-reference)，发布前的两道门与 severity 语义见[发布门禁与 preflight](./preflight-and-publish)。
