---
title: 错误排查
description: 按预检位置、错误码和运行阶段定位 Mod 问题
---

# 错误排查

> 适用范围：当前内容预检错误契约、Mod Maker 与 Workshop 状态；最近代码核对：2026-10-09；页面类型：排错参考。本页按主题排查；全部 47 码的三态总目录与编辑器提示对照见[错误码总目录](./error-codes)。

## 先确定问题在哪一层

1. **编辑器实时校验**：当前单元格、主键、关系或专用编辑器问题。
2. **Mod Maker 保存预检**：对当前编辑状态和即将写入的输出快照执行硬检查；失败不能绕过。
3. **Workshop 目录预检**：扫描已落盘的整棵目录、严格 Manifest、注册文件、链接/特殊文件和客户端资源；Mod Maker 保存预检通过后，仍可能在这里失败。
4. **预览问题**：前端 Mock、预览弹窗或资源目录准备失败。
5. **本地游戏问题**：启用集合、重启、客户端资源索引、协议或回退。
6. **Workshop / 服务端问题**：订阅下载、上传、服务器拉取、审核、场景选择和真实业务规则。

不要跨层猜测。例如 CSV 必填错误应在 Mod Maker 修复，不应通过重新订阅 Steam 解决。

`mod-toolkit validate` 还会报上传 `preflight` 不包含的作者诊断。兵种图的孤儿政权、孤儿兵种、无法识别的复合键、未映射的非 WebP 图片和被显式映射遮蔽的文件均为 `asset-reference` error：按完整 `arm_configs`/`ship_configs.code_name` 先判通用图，再判 `<regime code_name>_<troop code_name>.webp` 变体；核对 Mod 与原版实体，跨格式图片用 `Config/assets.json` 将有效 WebP 源键映射到图片文件。势力变体图缺失本身合法。省份浮层请求键是 `dynamic-assets/province-overlays/<code_name>.webp`。

1.6.0 官方底图已内含旧版内置地名。沿用底图且覆盖 GeoJSON 时，`validate` 对精确匹配旧版的 `text` 给 `asset-reference/legacy-map-text-overlap` warning；移除该旧文字，或提供有效的 `home-map-assets/base/ming-map.webp` 自定义底图。新文字不因此受限。此诊断不进入上传 `preflight`。

`quality-gate` 也可能报 error：`duplicate-ruler` 表示同一真实 `current_ruler` 被多个政权使用，应回表核实并修正。“纷争内斗”是无单一统治者时的精确占位值，予以豁免；其它真实人物不因占位规则而豁免。未裁定 issue 清零才算质量门通过。

## 如何阅读内容预检

先记录错误码和原因，再看相对路径、表、数据行、物理行、列、实际值和限制。数据行从 metadata 后第一条数据算起；物理行会受多行引用单元格影响。修复一个根因后重新检查，因为重复主键或缺失上游记录会引发多条下游错误。

若错误只在 Workshop 选择目录后出现，先检查它是否来自保存侧不扫描的旧残留文件、链接、`Config/assets.json`、严格 Manifest 或现存媒体；不要因为 Mod Maker 的保存面板是绿色就忽略目录级问题。

## 根目录与 Manifest

| 错误码 | 常见原因 | 修复 |
|---|---|---|
| `root-invalid` | 选到父目录、不可读目录或不是普通目录 | 重新选择直接包含 Manifest 的 Mod 根目录 |
| `mod-oversize` | 整个 Mod 目录超过 1000MB | 删除残留备份或缩小资源，使未压缩总量不超过限制 |
| `manifest-missing` | 没有 `manifest.yaml` | 用当前 Mod Maker 完整保存一次 |
| `manifest-invalid` | 旧扁平格式、版本/身份字段错误、多余叶节点或不安全路径 | 回 Mod 信息和保存流程重建，不手改 `files` 绕过 |
| `manifest-oversize` | 清单异常膨胀 | 检查重复注册和手工内容，重新生成 |
| `file-oversize` | Manifest 注册文件超过所属表档位或默认 50MB 上限 | 按错误给出的 actual/limit 缩小文件，不要只改后缀 |
| `registered-file-missing` | Manifest 指向已移动或删除的文件 | 恢复文件或重新保存更新清单 |
| `registered-file-type` | logical name 使用错误后缀 | 恢复约定的 CSV/JSON/MD/TXT 类型 |
| `registered-file-duplicate-physical` | 多个 logical name 指向同一文件 | 让每个注册项使用自己的规范路径 |

Manifest 路径含盘符、前导 `/`、反斜杠、冒号、空段、`.` 或 `..` 时会被拒绝。

## CSV 语法与 metadata

- `csv-syntax`：引号未闭合、双引号未转义或记录结构损坏。优先用备份恢复，再由 Mod Maker 保存。
- `csv-column-count`：某条记录单元格数与表头不同。检查含逗号或换行的单元格是否正确引用。
- `csv-metadata`：前四条逻辑记录缺失、错位或字段名不匹配。不要只补一行表头；用当前版本重建完整 metadata。
- `csv-row-limit`：数据记录数超过该表的 100、300 或 500 行档位。按限制页缩减内容；四条 metadata 不计入数据行。
- `csv-cell-limit`：至少一个单元格超过 10,000 Unicode code points。按错误中的表、数据行和列拆分或精简内容。
- `csv-import`：mod-toolkit `validate` 报出的导入层诊断（**不拦截上传**，warning 级）：`missing-table` 表 CSV 缺失（如 recommend_base.csv 未建，游戏按空表处理——用 `mod-scaffold --table` 生成骨架并登记）；`table-misplaced` 表放错 group 目录（如 ministers.csv 放进 global/，按 `expected` 参数移回归属路径）；`metadata-block` 缺或不足 4 行元数据块；`duplicate-header` / `empty-header` 表头列名重复或为空；`required-column-missing` / `column-default-filled` 缺必需列或缺列已按默认值处理；`ragged-record` 记录单元格数与表头列数不齐；`cell-invalid` 单元格类型 / JSON 错（指向单例表 finances 的行选择器已自动移除）。未知列保持 `csv-schema/unknown-column`，不在此码。
- `file-invalid-utf8`：文件是 GBK/ANSI 或包含损坏字节。以 UTF-8 重新保存。
- `file-changed`：检查后、写入前文件被外部程序修改。关闭外部编辑器，重新加载和保存。

## 字段、主键与外键

`csv-schema` 下常见原因：

- `required-column-missing` / `unknown-column`：缺少当前必填列，或包含当前 Schema 未定义的列；
- `required-value-missing`：当前必填单元格为空；
- `invalid-type` / `invalid-format` / `invalid-enum`：类型、结构或枚举值错误；
- `below-minimum` / `above-maximum`：数值超范围；
- `primary-key-missing` / `primary-key-duplicate`：主键为空或表内重复；
- `foreign-key-missing`：引用值在目标表中不存在；
- `event-key-duplicate`：相关事件表的 `code_name` 跨表重复。

先按[数据表参考](./tables/)核对当前字段，再从第 0 层向第 4 层补上游数据。不要把 `null` 字符串、空格或虚构 ID 当成占位值。

## 立绘表额外行

- `portrait-name-invalid`：额外行名称为空或含首尾空格、以 `数字_` 开头、不是小写字母开头的小写字母/数字/下划线组合、超过 64 个字符，或与两表派生立绘名重复。回立绘表编辑器修改名称。
- `portrait-name-duplicate`：多条额外行使用了相同名称。保留一条，其余改名。

派生行由 `ministers` 与 `recommend_ministers` 自动并入，只读不可改名；额外行与派生名同名时会被并入只读行，删除对应大臣也不会恢复该额外行。

## 事件、决议与国策

- `fixed-turn-event-invalid`：固定回合事件必须在顶层条件数组包含 `type: "turn"`、`judge_type: "equal"`、有限整数 `value`。
- `decision-structure-invalid`：回到决议详情检查条件、重复策略、效果和超事件关系。
- `perk-relation-invalid`：前置或互斥目标不存在、自引用、互斥不对称或关系冲突；使用国策树编辑器修复。

预览能展示事件不等于真实条件正确。本地 Mod 仍使用官方服务端场景；只有 Workshop 场景就绪后，才能用正反两组局面验证真实条件。

## JSON 与文本

| 错误码 | 修复 |
|---|---|
| `json-syntax` | 删除注释和尾逗号，补齐双引号/括号 |
| `yaml-syntax` | ScenarioMeta.yaml 无法被严格解析为 YAML；修正缩进、引号、冒号等 YAML 语法，用 YAML 校验器或按保存预检报错定位 |
| `json-schema` | 结构校验：`pg_table_mapping` 的顶层类型、字段类型、snake_case 与别名唯一性，以及未知表键拒收（顶层键必须是 schema 内的 PG 表名）；`game_config`（年号段/季节循环/数字制式/回合模板）、`province_schema`（命令字段与 defaultValue 类型匹配）、`page_operation_guides`（path 固定四值且不重复、markdown 非空）、`key_events`（turn 数字、event 非空）、`global_config`（两必填非空字段）、`minister_positions`（分组/职位字段类型与 official:null）、`minister_chat_presets`（trigger/prompt 非空）、`court_chat_quick_messages`（字符串数组项非空）、`province_view_config`（覆盖片段：version 必为 1、视图 id 限官方七视图、rule 按 kind 校验 colors/ranges 与区间连续性）、`map_behavior_config`（version 必为 1、maxScale 限 1–8、未知顶层键拒收）；跨表引用失配同码拦截（`json-fk-target-missing`：`global_config.main_regime_code` 不在 `regimes.code_name`、地图省份要素 `code_name` 不在 `provinces.code_name`——改名后同步对端即修复；`main_regime_code` 失配在编辑器保存时同样被拦，全局配置编辑器会以下拉“未匹配”标记直接指认）——字段拼错、必填键漏写、类型不对都会在此被拦，按 message 指到的字段修复 |
| `json-entry-limit` | 把条目减到 1,000 以内，不拆成未注册文件规避 |
| `text-char-limit` | 精简单个 Markdown 层文件、TXT 或 YAML 到 20,000 code points 以内；mod-toolkit `validate` 在超过 18,000（硬顶 90%）时会提前报 `approaching-limit` 余量预警（warning，不拦截上传），收到即应着手精简 |
| `prompt-layer` | mod-toolkit `validate` 报出的 setting 层巡检（**不拦截上传**，warning 级）：`setting-missing` 声明 ScenarioMeta 的剧本 mod 缺 setting 层文件（聚合单条，`missing` 参数列全缺失名单——用 `mod-scaffold --prompt` 生成 10 个骨架并分层登记；纯数据 mod 不报）；`setting-empty` setting 文件存在但空白/仅空白行（填写本剧本内容或删除该文件）。setting 层缺失/空白时 AI 将按原版世界观推演 |
| `prompt-render-residual` | validate 报出的渲染模拟（warning；廷议条目 **error**）：`static-layer` static 条目存在 `{{ }}` 变量首名不可解析且无 `\| default()` 兜底——服务端 fail-open 会把未渲染原文原样发给模型；`court-discussion` 廷议模板（严格渲染）同款残留即 error。改用可解析变量名（ScenarioMeta 顶层键 / time_label / time_note；廷议另有 11 个调用点变量）或补 default。变量宇宙 = mod 自己的 ScenarioMeta（缺失回退原版） |
| `scenario-meta-unknown-key` | validate 报出（warning，宽容度 ≥ 服务端 extra=allow）：`unknown-key` 顶层键不在 13 个 schema 字段 ∪ 原版键集——rome/sanguo 式增键合法不拦，但需确认后端有消费方，否则是死配置 |
| `prompt-structure` | preflight 报出的分层结构闸（**error**，镜像服务端写前校验）：`suffix-mismatch` 键名不以 `.{layer}` 结尾（运行时按后缀识别层，二者必须一致）；`wrong-subdir` 层文件不在 `prompt/{layer}/` 子目录（移动文件并同步 manifest path）；`layer-outside-prompt` layer 字段出现在非 prompt 组（删掉 layer）；`yaml-layered` YAML 类 prompt 声明 layer（层间分隔注释会破坏 yaml.safe_load，删 layer 保持 flat） |
| `scenario-meta-schema` | preflight 报出的元信息 schema 闸（**error**，镜像服务端审核阶段 ScenarioMetaConfig）：`field-type` 13 个字段严格类型不符（字符串字段不接受数字、start_year 必须整数、display_terms 必须字符串映射）；`seasons-not-four` seasons 必须恰好 4 个标签（1 年 = 4 季换算）。改值或改类型即消除 |
| `prompt-variable-missing` | 平台声明为必填的变量在合成文本中未被引用；在设定层/示例层补回该变量的 `{{ 变量名 }}` 占位符（error，阻断） |
| `prompt-variable-undeclared` | 合成文本中出现平台未声明的 `{{ 变量名 }}`，多为拼写错误；核对变量名拼写或改为普通文本（warning 级：编辑器保存预检确认后可放行；但上传门禁零容忍——issues 未清空即拒收） |

官职 JSON 的分组结构、职位字符串字段与 `official: null`，全局配置标准键 `edict_opening_phrase`、`main_regime_code` 非空，以及 `main_regime_code` 对 `regimes.code_name` 的跨表匹配（`json-fk-target-missing`），现均由上传预检的 `json-schema` 校验把关；其中全局配置的两项（必填非空与跨表匹配）在编辑器保存时同样拦截，主政权经政权表下拉选择即天然匹配；PG 别名必须小写 snake_case 且唯一、未知表键拒收。仍不在阻断范围的：官职的空名称、重复 `key`、重名职位——使用专用编辑器、人工核对和预览回退结果，不能只看预检通过结果。若预览主政权正确但游戏配置回退，检查是否只写了当前契约定义的两个标准键。

## ScenarioMeta.yaml

`prompt/ScenarioMeta.yaml` 是 YAML 迁移后的剧本元信息（替代原 ScenarioMeta.md），也是 `prompt/` 下唯一不分层的文件——13 个 Markdown 逻辑提示词按 `prompt/{setting|rules|example}/<逻辑名>.md` 分层。它经过两道闸门：Mod Maker 保存预检（`yaml-syntax` 硬阻断）→ Workshop 目录预检（同码）。

规则：

- 必须可被严格解析为 YAML；语法错误在保存预检即报 `yaml-syntax`，不能绕过。
- 单文件同样不得超过 20,000 Unicode code points，超限报 `text-char-limit`。

常见错误：

- 同级键缩进不一致；
- 值含冒号或特殊字符未加引号；
- 使用 Tab 缩进（YAML 只允许空格）。

最小正确示例：

```yaml
protagonist_title: 崇祯皇帝
ruler_name: 朱由检
honorific: 陛下
era_name: 崇祯
```

变量与显示名口径见[文本与提示词](../advanced/texts-and-prompts.md)。

## 图片与资源映射

| 错误码 | 含义 |
|---|---|
| `asset-config-invalid` | `Config/assets.json` 结构错误 |
| `asset-path-invalid` | 绝对路径、父目录、短目标路径、目录或越界 |
| `asset-extension-unsupported` | 后缀不在允许资源族中 |
| `asset-family-mismatch` | source 和 target 跨图片/音频/视频/字体/JSON 资源族 |
| `asset-duplicate-source` | 同一 source 重复映射；保留第一条有效项 |
| `asset-file-oversize` | 单文件超过该资源族上限 |
| `asset-image-invalid` | 位图无法解析 |
| `asset-image-pixels` | 位图超过 64MP |
| `asset-json-invalid` | 资源 JSON 无法严格解析 |
| `asset-reference` | mod-toolkit `validate` 报出的跨文件引用完整性问题（**不拦截上传**）：`mirror-diverged` 注册 JSON 双副本内容不一致（以一份覆盖另一份）；`file-unregistered` 注册命名空间（pg/、mongo/、global/、prompt/、text/ 及根级注册 JSON）内容文件未登记 manifest——未注册文件不会被游戏加载，用 `mod-scaffold --table` 增量登记或手改 `files`；`asset-missing` 新增实体（不在原版同名表）的约定资源文件缺失（事件图、兵种图、技能图标、省份浮层、立绘四件套、政权驻军图标 `dynamic-assets/garrison-icons/<code_name>.webp`）——运行时静默回退官方资源，把文件放进 client-assets 对应目录或经 `Config/assets.json` 显式映射；原版已有实体回退属合法语义不报 |
| `derived-table` | mod-toolkit `validate` 报出的派生表一致性问题（**不拦截上传**）：`derived-file-missing` 大臣两表有 code_name 但 `global/minister_assets.csv` 缺失（运行时静默回退官方立绘注册表），或举荐两表有匹配行但 `global/recommend.csv` 缺失（回退官方举荐数据）；`derived-diverged` 文件与源表现算派生不一致（缺失/多余/顺序错/行内容被手改）——分别用 `derive-tables <目录>` 与 `derive-tables <目录> --table recommend` 重建修复 |
| `field-definitions` | mod-toolkit `validate` 报出的字段元数据覆盖对账问题（**不拦截上传**，warning 级）：mod 的 `client-assets/field-definitions.json` 稀疏覆盖 ↔ 官方基线（51 Schema）对账——`unknown-schema` 覆盖的 Schema 名不在基线、`unknown-field` 字段名不在基线该 Schema 字段集、`unknown-property` 属性键超出 label / description / type / ref（三者运行时只忽略并打诊断，写了等于没写；用 `field-definitions` 工具查有效 Schema 名与字段树后修正）、`invalid-shape` 覆盖文件 root / schemas / fields 非对象（root / schemas 形状非法时整份覆盖运行时回退官方基线）；稀疏覆盖合法不报覆盖不全，label / description 文本不校验 |
| `path-link` / `path-special-file` | 路径指向链接或非普通文件 |

统一把示例路径改成 `/`，确认 target 是完整 `client-assets/...` 普通文件。修复映射后仍要重启打包版，重新建立活跃资源索引。

## 样式（custom.css）

Workshop 整目录预检对根目录的实际 `custom.css` 文件执行 1MB 大小检查，超限即报 `custom-css`（error，阻断上传）：

| 错误码 | 常见原因 | 修复 |
|---|---|---|
| `custom-css` | 超过 1MB（1,048,576 UTF-8 字节，运行时整文件拒绝注入） | 按 UTF-8 字节把文件精简到 1,048,576 字节以内后重新预检 |

小写 `px` 不是缺陷：运行时按 1920×1080 基准把小写 `px` 换算为响应式 `calc()`（大写 `PX` 豁免），属于正常机制。`@import`、外部 `url()` 与选择器锚定等写法层问题不在预检范围——运行时安全编译会分别剥离、中和、整条丢弃，锚点、稳定名与保护页面的完整规则见[自定义样式](../resources-and-ui/custom-css)，用预览验证实际效果。

## 浏览器、保存与预览

- 无法选择目录：使用游戏内入口或支持 File System Access API 的 Chromium 环境，并重新授予权限。
- 普通浏览器点击预览提示“请从游戏内打开”：这是当前能力边界，不是弹窗权限故障；改从游戏设置页进入 Mod Maker。
- 恢复会话失败：目录被移动或权限失效；从首页重新选择真实目录。
- 打开/Fork 后部分内容变空或恢复原版：缺失源 CSV 会按空表加载，缺失或无法解析的注册 JSON 会使用默认值，缺失提示词/TXT 会回退当前原版；立即停止保存，对照完整备份修复源目录或改用 Fork 保存到新目录。
- 关闭或刷新前没有出现未保存提示：当前没有覆盖整个编辑器的统一离开保护，顶部撤销/重做也只覆盖表数据；返回编辑器检查各工具状态并主动完整保存。
- 保存失败：查看第一条内容问题；权限错误时重新授权，不要换到父目录。
- 游戏内保存 `custom.css` 提示超过 1MB：按 UTF-8 字节把文件缩到 1,048,576 字节以内再完整保存；样式在结构化快照之后写入，这次失败可能已经留下部分更新目录，不要直接上传。独立浏览器路径同样有这条 1MB 写入拦截，保存失败时同样处理。
- 预览窗口没有打开：游戏内预览经主进程打开、不依赖浏览器弹窗权限；按“重试预览”再试，仍失败时先保存并确认 Mod 根目录选择正确。
- 预览资源未准备：保存一次，检查根目录锚点和预览目录注册。
- 预览显示 Mod 数据、本地游戏仍显示官方业务数据：这是本地 `local_` Mod 的正常边界；不要反复导入修复。
- 预览中的客户端资源正确、本地打包版错误：检查保存状态、资源路径、映射、重启和协议回退。
- Workshop 预检通过但游戏没有样式：整目录预检只排除 `custom.css` 超过 1MB（见上文「样式」节）；`@import`、外部 URL、未锚定选择器等写法会被运行时安全编译剥离、中和或整条丢弃——无样式时先检查 CSS 语法、选择器锚定、稳定名拼写与是否命中受保护页面。
- Workshop 场景中的真实规则与预览不同：预览只负责有限 Mock；以服务端结果为准，按条件、效果和场景状态排查。

## 本地 Mod、活跃快照与重启

- 导入提示缺 Manifest：选择错层级或未完整保存；按对话框指引重新选择并导入，或返回 Mod Maker 保存一次。
- 导入弹出「该 Mod 的资源未通过校验」确认框：导入在托管副本上执行与上传同源的资产校验（`Config/assets.json` 映射、扩展名、字节与像素预算、驻军图标）；「无视风险继续安装」会在复检后完成导入，「取消导入」清理临时副本——这些问题在上传 Workshop 时同样会被拒，发布前必须修复。
- 导入提示已存在同名 Mod：游戏不会覆盖相同 `manifest.id` 的托管副本；确认原工程已有备份，删除旧本地条目后重新导入。
- 启用后不生效：确认卡片条目和目录，按顶部“待重启”横幅重启。
- 修改后客户端内容没有更新：本地条目运行 `userData/mods/<manifest.id>/` 中的托管副本，不会同步 Mod Maker 原工程；删除已有条目、重新导入最新原工程、启用并重启。
- 修改 CSV 后本地游戏仍是官方数据：不要把它当作缓存问题；CSV 先在预览中检查，真实数据等待 Workshop 服务端场景。
- 导入因链接或特殊文件失败：移除 Mod 目录中的符号链接、目录联接和非普通文件后再试；导入不会跟随链接复制目录外内容。
- 删除本地 Mod：永久删除游戏管理目录里的托管副本，不会删除外部原工程；仍需先确认原工程有备份。删除已启用条目会先弹确认框，确认后改动保存并进入待重启横幅。
- 本地资源生效但业务场景仍是官方：本地 `local_` Mod 按设计使用官方场景。

## Steam 下载与更新

- 未安装/需要更新：使用“下载/更新”，等待排队、下载和安装完成。
- 等待 Steam 或“Steam 响应较慢”：下载请求被接受但状态尚未传播；等待约 60 秒仍未进入传输时会出现响应较慢提示，可重试并检查 Steam 在线状态。
- 下载阶段约 5 分钟没有字节进展时，卡片会显示“下载缓慢”；排队或安装阶段可能仍保留原阶段文字。保持 Steam 在线，稍后重试。
- 状态不可用：检查 Steam 客户端与登录；本地 Mod 仍可管理。
- “检查更新”后开始下载：这是正常行为，该动作会请求 Steam 下载而非只读查询。
- 前台 30 秒没有变化：按钮进入 5 分钟冷却，后台仍观察到冷却结束。
- 更新完成但内容未变：已启用 Mod 必须重启。
- “检查更新”后客户端资源仍未更新：可“取消订阅”后重新订阅，让 Steam 重新拉取完整客户端文件；此兜底只用于客户端资源滞后，服务端内容问题不适用。

Steam“就绪”只证明本地安装目录可读，不等于九种业务服务端状态已经“已上架”。带剧本型标签的活跃 Workshop Mod 会在启动时直接使用 Item ID 作为 `scenarioId`；若服务端仍在拉取、审核、待上架或失败，应回上传管理页检查状态原因，而不是反复下载客户端文件。

## 上传、协议和审核

- 创建按钮不可用：补齐名称、描述、内容目录并等待预检通过。
- 预览图错误：仅支持 png / jpg / jpeg 且不超过 1MB（gif / webp 会解码失败）。
- 要求接受协议：这一轮提交已经停止，内容和服务端注册不会自动续传。接受后重新进入“上传模组”刷新，先检查是否留下未上传的 Steam 条目；当前没有一键续接首次注册，记录 Item ID 后再决定删除空条目重建或联系维护者。
- 创建 Steam 条目后提示服务端注册失败：Steam 侧可能已经留下空条目。不要连续创建多个；记录 Item ID、刷新两侧状态，并在确认无需要保留的关联后处理孤立条目。
- 更新后文件没变：编辑弹窗不会保留上次的内容目录；文件有变化时必须重新选择最新 Mod 根目录并等待预检。
- 提示“通知后端上传完成失败”：Steam 内容已经上传，但服务端流程可能不会推进；保留 Item ID 和工程，待服务恢复后在同一条目重新选择内容目录并更新，以再次触发通知，不要另建条目。
- 资源拉取/校验失败：查看状态原因，重点检查 Manifest、注册文件和内容目录。
- 审核未通过：阅读状态原因；服务端提供 URL 时打开“审核报告”。没有按钮不一定是客户端故障。
- 已下架：当前是终态，保留工程和条目信息，按运营流程处理。

## 不丢数据的恢复顺序

停止继续覆盖 → 复制当前目录 → 保留原件只读 → 用当前 Mod Maker Fork 副本 → 按第一条根因修复 → 保存到新目录（Fork 首次保存即完整复制源目录，样式、显式映射与未注册媒体都会随带，无需补拷）→ 预览支持的数据 → 本地导入并重启验证客户端 → 更新 Workshop → 服务端场景就绪后回归真实玩法。不要通过删除 Workshop 条目、清空 Steam 缓存或重写 Manifest 掩盖内容问题。
