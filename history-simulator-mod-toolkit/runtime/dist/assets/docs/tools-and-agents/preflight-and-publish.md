---
title: 发布门禁与 preflight
description: 编辑器保存门禁与 Workshop 上传门禁两道门、severity 语义与零容忍
---

# 发布门禁与 preflight

> 适用范围：内容预检契约 v2 与 47 个顶层错误码、mod-toolkit `preflight` 内核实现；最近代码核对：2026-10-09；验证层级：工具输出与游戏上传门禁逐码一致（同一内核实现）。

## 两道门：检查对象与判定都不同

Mod 从编辑到上架要过两道独立的预检，不要混为一谈：

| | 编辑器保存门禁 | Workshop 上传门禁 |
|---|---|---|
| 检查对象 | 当前编辑状态与即将写入的输出快照 | 选择内容目录后落盘的整棵目录 |
| 执行位置 | Mod Maker 保存流程（[保存与内容预检](../validate-and-publish/save-and-preflight)） | 上传 / 更新条目前的整目录预检（[发布到 Workshop](../validate-and-publish/workshop)） |
| severity 语义 | **分级**：error 级硬拦保存；warning 级弹确认框，作者确认后可以继续保存 | **零容忍**：对任何 issue 一律拒收——`valid` 等价于 issue 清零，warning 同样拦截 |
| 也可由谁执行 | —（编辑器内部） | `mod-toolkit preflight`（与本门禁同一内核实现，逐码一致） |

保存绿灯不等于上传绿灯：保存门禁只看编辑状态与快照，整目录里由旧残留文件、链接、严格 Manifest、资源映射带来的问题只会在上传门禁暴露。反过来，不经编辑器的目录（纯 agent / 手工制作）只面对上传门禁这一道。

## preflight 检查什么

`preflight <Mod 目录>` 对整棵目录执行发布门禁检查：

- **manifest 严格校验**：`manifest.yaml` 的身份字段、注册清单与未知字段（手改 `files` 绕不过）；
- **注册文件**：清单声明的每个 logical name 都要存在、类型正确、不重复指向同一物理文件；
- **CSV 限制**：行数档位（100 / 300 / 500 行按表）、单元格 10,000 码点、语法与 metadata 块；
- **JSON / YAML 语法**与条目上限；`game_config` / `province_schema` / `page_operation_guides` / `key_events` / `global_config` / `minister_positions` / `minister_chat_presets` / `court_chat_quick_messages` / `province_view_config` / `map_behavior_config`，另含 `pg_table_mapping` 映射结构与 `map_province` GeoJSON——共十二个注册 JSON 的**字段结构校验**（`json-schema` 码，字段拼错、必填键漏写、类型不对都会被拦）；`Config/assets.json` 显式映射合法性；
- **client-assets 资源预算**：扩展名白名单、单文件字节上限（按资源族 4MB–256MB）、图片 ≤ 64MP、驻军图标目录的 webp 特例；
- **立绘表额外行**与 **prompt 合成变量**（`{{ 变量名 }}` 的缺失 / 未声明）；
- **custom.css 大小检查**：1MB 上限（`custom-css` 错误码）。

## severity 与「过预检 = 可上传」的确切含义

每条 issue 都带 `severity`（`error` / `warning`）。47 个顶层错误码中绝大多数是 error 级，码级 warning 共 7 个，按落位分两类：`text-char-limit`、`prompt-variable-undeclared` 两个码属于上传阻断分区——warning 级但上传门禁零容忍同样拦截（两者性质是质量 / 边界提示而非结构违规，在**编辑器保存门禁**里可以确认后放行）；`field-definitions`、`csv-import`、`prompt-layer`、`prompt-render-residual`、`scenario-meta-unknown-key` 五个 warning 级码是 mod-toolkit `validate` 独有的对账 / 导入诊断 / setting 层巡检 / 渲染残留 / 未知配置键提示（不进入上传门禁，见[错误排查](../reference/troubleshooting)）；error 级的 `asset-reference` 与 `derived-table` 同样只在 `validate` 产出、不拦截上传。

但**上传门禁对任何 issue 零容忍**：warning 同样拒收。所以正确的目标不是「清零 error」，而是「清空 issues」。工具与游戏说同一套 severity 语言，你可以按 severity 排修复优先级，但不能按 severity 决定修不修。

由于 `mod-toolkit preflight` 与游戏上传门禁是同一内核实现、逐码一致，**工具侧 `valid: true` 即代表上传预检会放行**（前提是工具版本与游戏版本同源发布）。游戏内的上传预检在 worker 线程执行并有 30 秒超时——超时按系统错误拒收，重试即可。每条 issue 都带稳定 `code`（错误码）、相对路径与中文 message，修复指引见[错误排查](../reference/troubleshooting)。

## 与 validate 的关系

`validate` 覆盖 CSV 数据面（主键、类型、外键、专项规则）、导入层诊断（缺表、表放错 group 目录、元数据块 / 表头 / 列数 / 单元格问题，`csv-import` 码 warning 级）、跨文件引用完整性（注册 JSON 双副本镜像一致、注册命名空间文件登记，`asset-reference` 码——这些一致性检查不拦截上传）与 prompt 层巡检（setting 层缺失 / 空白 `prompt-layer` 码、文本接近 20,000 码点硬顶的 `text-char-limit/approaching-limit` 余量预警，均 warning 级不拦截上传），`preflight` 是整目录门禁——两者是包含与被包含之外的互补关系，工作流契约要求四件套全过（`validate` 0 error + `preflight` valid + `quality-gate` 未裁定 issue 为 0 且已裁定无 error + 缺表 0），缺一不可。CI 里可以把两条命令按 `validate` → `preflight` 顺序串联，任一退出码非 0 即失败（1 = mod 不合法，2 = 工具或用法故障），详见[工具参考](./tools-reference)的退出码约定。

## 检查集对照：四道门各查什么

| 检查 | `validate` | `preflight`（= 上传门禁内核） | 编辑器保存预检 | 服务端上传门禁 |
|---|---|---|---|---|
| CSV 数据面（csv-schema 族：必填 / 类型 / 主外键 / 专项规则） | ✔ error 级 | ✔ 同码入目录门禁 | ✔（自有 6 类校验器，与内核有意分叉） | 契约未披露 |
| CSV 结构项（`csv-syntax` / `csv-column-count` / 行数档 / 单元格上限） | 仅 `csv-syntax`；列数不齐走 `csv-import` 诊断 | ✔ | ✔（自有实现） | 契约未披露 |
| 导入层诊断（`csv-import`：缺表 / 放错目录 / 元数据块 / 表头 / 单元格） | ✔ warning，不拦截上传 | — | 部分（导入回填告警） | — |
| manifest 严格契约 / 注册文件 / assets.json 映射 / 资源预算 / 注册 JSON 结构 | — | ✔ | ✔ | ✔ |
| 跨文件引用完整性（`asset-reference`）/ 派生一致性（`derived-table`） | ✔，不拦截上传 | — | 派生保存时现算 | — |
| field-definitions 对账 / `prompt-layer` 巡检 / 字符余量预警 | ✔ warning，不拦截上传 | — | — | — |
| 必需逻辑文件已声明（`requiredLogicalFiles`） | — | —（契约 pending，未实现） | — | 契约未披露 |
| 判定语义 | error 清零 + warning 知情 | issues 清空（warning 同拦） | error 硬拦 / warning 确认放行 | 拒收 |

服务端列只写客户端契约可见的已知项；「契约未披露」表示客户端看不到该层是否另有等价检查，不要把客户端全绿当作服务端无别的检查（`requiredLogicalFiles` 的启用时间表属三方确认问题 2 的讨论范围）。

此外还有**第五道门：本地导入的资产校验确认门**。导入本地 Mod 时，游戏会在托管副本上执行与上传门禁同源的资产校验（`Config/assets.json` 显式映射、client-assets 扩展名、字节与像素预算、驻军图标）；与上传门禁的零容忍不同，发现问题以「无视风险继续安装 / 取消导入」确认框放行，确认后还会复检 staging 完整性与目标冲突（见[本地导入和客户端测试](../validate-and-publish/local-testing)）。

两个高频盲区，别再踩：

- **`csv-column-count` 只在 preflight 报**：列数不符的行在 validate 里先报 `csv-import/ragged-record` 诊断（warning，每表前 5 条 + 汇总），并可能伴随一串 `required-value-missing`（必填字段被补空）——别去补字段，先修该行的列结构。
- **validate-only 码（`csv-import` / `prompt-layer` / `asset-reference` / `derived-table` / `field-definitions` / `prompt-render-residual` / `scenario-meta-unknown-key` 七码）不拦截上传**：但工作流契约仍要求 validate 0 error——「不拦截」不等于「不重要」，它们是本地唯一对整表缺失、死覆盖、资源未登记等问题发出信号的检查。
