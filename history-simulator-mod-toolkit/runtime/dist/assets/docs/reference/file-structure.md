---
title: Mod 文件结构
description: 完整了解 Manifest、CSV、JSON、提示词、文本、资源和样式文件
---

# Mod 文件结构

> 适用范围：当前内容契约 v2、Manifest v1；最近代码核对：2026-10-14；页面类型：文件参考。

## 完整目录结构

```text
my-mod/
├─ manifest.yaml                         # 必需，结构化文件中最后写入
├─ global_config.json                    # 全局配置权威文件；Manifest 注册
├─ pg-table-mapping.json                 # Manifest 注册 JSON
├─ custom.css                            # 可选，已启用 Mod 自动编译注入
├─ pg/                                   # PG 源 CSV
│  ├─ regimes.csv
│  └─ ...
├─ mongo/                                # Mongo 源 CSV
│  ├─ normal_events.csv
│  └─ ...
├─ global/                               # 全局源 CSV、派生 CSV 与官职 JSON
│  ├─ minister_positions.json
│  ├─ game_config.json                   # 历法的 Manifest/服务端副本
│  ├─ recommend.csv                      # 自动生成
│  ├─ minister_assets.csv                # 派生+额外行（立绘表编辑器）
│  └─ ...
├─ prompt/                               # 13 个 Markdown 逻辑提示词（分层）+ 1 个不分层 YAML
│  ├─ setting/                           # 设定层：世界观、人物、语气，创作者可编辑
│  ├─ rules/                             # 规则层：机制与协议，平台统一维护（服务端不落库、仅展示）
│  ├─ example/                           # 示例层：可选 few-shot，可为空
│  └─ ScenarioMeta.yaml                  # 剧本元信息（YAML 整体解析，不分层）
├─ text/
│  └─ default_turn_0.txt                 # 开场文本
├─ client-assets/
│  ├─ game-config.json
│  ├─ global_config.json                 # 全局配置的客户端副本
│  ├─ key-events.json
│  ├─ minister-chat-prompt-presets.json
│  ├─ court-chat-quick-messages.json
│  ├─ page-operation-guides.json
│  ├─ field-definitions.json
│  ├─ map-data/
│  │  ├─ province-schema.json
│  │  ├─ ming_provinces.json
│  │  ├─ province-view-config.json
│  │  └─ map-behavior-config.json
│  └─ ...                                # 图片、音频、视频、字体等资源
└─ Config/
   └─ assets.json                        # 可选显式资源映射
```

Mod Maker 完整保存会输出 27 张源 CSV、2 张派生 CSV、4 个 Manifest 注册 JSON、11 个固定客户端 JSON、28 个 Markdown 提示词层文件（10 个设定层 + 12 个规则层 + 6 个示例层；示例层留空则省略写出）、1 个 YAML、1 个 TXT 和 Manifest，并写出编辑器待保存区中的已注册图片与样式草稿。绑定目录里原本存在的其他文件通常会保留，但它们不是上述注册输出的一部分；已退役的 vanilla 专属文本（`text/gsr_yuan_kills_mao.txt`、`text/gsr_jisi_incident.txt`，信息感知推演指令长文为官方内置内容）是个例外——保存时会从 mod 目录移除。

“Fork 另一 mod”首次保存会完整复制源目录，保留 `custom.css`、`Config/assets.json` 和未注册文件；随后用当前编辑器中的权威配置刷新两对副本，并把 `game_config` 的 Manifest 位置归一到 `files.global.game_config`。目标必须是独立空目录。

## 文件所在目录不等于验证方式

| 文件类别 | 制作期本地验证 |
|---|---|
| 23 张已接入预览适配器的源 CSV | 内容预检 + 游戏页面 Mock；可检查展示、关联、部分派生值和有限短流程 |
| `random_events_chain` | 事件链专用编辑器 + 内容预检；虽进入快照，当前没有页面消费者 |
| `fixed_turn_events`、`special_simulations`、`game_state_rules` | 普通主从表格 + 内容预检；当前不进入游戏预览快照 |
| `global/minister_positions.json`、权威 `global_config.json` | 支持预览页面 Mock 和编辑快照热更新；Preview 不读取客户端镜像 |
| `pg-table-mapping.json` | 编辑器 + 内容预检，不参与页面 Mock |
| `prompt/setting/*.md`、`prompt/rules/*.md`、`prompt/example/*.md`、`prompt/ScenarioMeta.yaml`、`text/*.txt` | 编辑器格式/变量/长度检查，不进入游戏预览 |
| `client-assets/`、`custom.css` | Mod Maker 预览；保存后可在本地打包版验证客户端真实覆盖 |

本地导入不会把根目录、`pg/`、`mongo/` 或 `global/` 中的数据写入服务端。真实数据库、条件、AI 与多回合行为只能在 Workshop 服务端场景就绪后验证。

## `manifest.yaml`

Manifest 位于根目录，`manifest_version` 固定为 `1`。Mod 身份四项都必填：

```yaml
manifest_version: 1
mod:
  id: my_mod_id
  name: 我的 Mod
  description: 一句话介绍作品
  author: 作者名
files:
  pg:
    regimes:
      path: pg/regimes.csv
  global:
    minister_positions:
      path: global/minister_positions.json
    game_config:
      path: global/game_config.json
  global_config:
    path: global_config.json
```

`mod.id` 长度 1～64，只能包含英文字母、数字、下划线和连字符。`files` 的叶节点允许 `path`、`required`、`layer` 三个键；当前只有 `prompt` 组的分层条目会携带 `required` 与 `layer`（见下文「提示词与文本」），其余组仍只有 `path`。按 `pg`、`mongo`、`global`、`prompt`、`text` 等分组，根文件可直接注册。

Manifest 路径必须是使用 `/` 的安全相对路径，不能含反斜杠、盘符、冒号、NUL、前导 `/`、空路径段、`.` 或 `..`。logical name 不能含斜杠，logical name 和物理路径都不能重复。正常制作不要手改 `files`；让 Mod Maker 在构建不可变输出快照时重建 Manifest，并在其他结构化文件之后写入它。

## CSV 通用格式

27 张源表和 2 张派生表都使用四条 metadata：

1. 英文字段名，同时是实际表头；
2. 中文显示名；
3. 字段说明；
4. 映射名；
5. 从第五条逻辑记录开始才是正式数据。

读取支持常见的 CRLF/LF 混用；规范保存统一输出 LF。单元格含逗号、双引号、CR 或 LF 时必须按 CSV 规则加引号，内部双引号写成两个双引号。不要按“物理文本行”直接拆 CSV，多行单元格会跨行。

全部源表路径和字段见[27 张数据表参考](./tables/)。`global/recommend.csv` 保存时生成、不可手工维护；`global/minister_assets.csv` 在立绘表编辑器维护（派生行只读、额外行可编辑），不作为普通源表。

## 29 个 CSV logical name 全清单

27 张源表按归属目录（manifest 的 group）+ 2 张派生表，全清单如下——不需要从原版数据反推：

| 归属目录 | logical name（27 张源表） |
|---|---|
| `global/`（4） | `player_attributes_init_table`、`recommend_base`、`recommend_ministers`、`super_events` |
| `mongo/`（9） | `decisions`、`fixed_turn_events`、`game_state_rules`、`initial_tasks`、`normal_events`、`perks_table`、`random_events`、`random_events_chain`、`special_simulations` |
| `pg/`（14） | `arm_configs`、`buildings`、`equipments`、`events_prediction`、`finances`、`land_tax_configs`、`ministers`、`parties`、`province_land_holdings`、`provinces`、`regimes`、`ship_configs`、`social_classes`、`troops` |

| 派生表（2，不可手工维护） | 生成方式 |
|---|---|
| `recommend` | 编辑器保存时由 `recommend_base` × `recommend_ministers` 按 code_name 联结生成 |
| `minister_assets` | 编辑器立绘表维护（派生行只读）；命令行用 `mod-toolkit derive-tables <目录>` 从大臣两表重建 |

**核对命令**：`mod-toolkit list-tables <mod 目录>`（MCP 名 `list_tables`）逐表打印行数与导入告警，末尾汇总 `共 27 表，缺失 N，导入告警 N`；`validate` 报告的 `contractCoverage` 字段则是结构化判据（`presentTables` / `missing` / `misplaced`，`missing` 为空才算数据面齐备）。清单齐全性由 `check:content` 从权威 schema 派生强制。

## 四个 Manifest 注册 JSON

| 路径 | 编辑器 | 核心结构 |
|---|---|---|
| `global/minister_positions.json` | 官职编辑器 | 分组数组；每个官职的 `official` 固定为 `null` |
| `global/game_config.json` | 历法配置编辑器生成的服务端副本 | 与权威 `client-assets/game-config.json` 保存时字节一致 |
| `global_config.json` | 全局配置编辑器 | 标准键 `edict_opening_phrase`、`main_regime_code` 均为非空字符串；主政权从政权表生成的下拉中选择，失配 `regimes.code_name` 会被保存与上传预检拦截 |
| `pg-table-mapping.json` | PG 表名映射 | 顶层对象；别名小写 snake_case 且全局唯一 |

保存时会重建注册关系。统一保存/上传预检会检查四者的 JSON 语法、大小和条目数，并对 PG 表名映射执行专门的字段与唯一性检查；官职和全局配置的字段结构也由上传预检的 `json-schema` 校验把关（见 13.2 / 13.3），但非空与唯一性不在阻断范围。官职应回专用编辑器维护，全局配置的主政权在编辑器中从政权表下拉选择、失配 `regimes.code_name` 时保存与上传预检都会拦截（`json-fk-target-missing`），无需人工核对。

预览窗口会消费官职 JSON 和全局配置，用它们构造官职、政权、省份和总览等页面 Mock。PG 表名映射不参与页面 Mock。

## 十一个客户端 JSON

固定路径为：

- `client-assets/map-data/province-schema.json`
- `client-assets/game-config.json`
- `client-assets/global_config.json`
- `client-assets/key-events.json`
- `client-assets/minister-chat-prompt-presets.json`
- `client-assets/court-chat-quick-messages.json`
- `client-assets/map-data/ming_provinces.json`
- `client-assets/map-data/province-view-config.json`
- `client-assets/map-data/map-behavior-config.json`
- `client-assets/page-operation-guides.json`
- `client-assets/field-definitions.json`

## 提示词与文本

`prompt/` 下注册 14 个逻辑提示词：13 个 Markdown 逻辑名 + 1 个不分层 YAML（`ScenarioMeta`，剧本元信息）。13 个 Markdown 逻辑名按职责拆成一至三个**层文件**，磁盘形态为 `prompt/{setting|rules|example}/<逻辑名>.md`：

- **设定层（setting）**：世界观、人物、语气，创作者可编辑；
- **规则层（rules）**：机制、协议与禁令，平台统一维护；服务端不落库、不使用 rules 层文件（仅供展示），本地与服务端均不再校验其内容。编辑器中只读，保存时自动携带官方基线字节；
- **示例层（example）**：可选 few-shot；留空时保存省略写出、Manifest 不登记。

| 逻辑名 | 层组合 |
|---|---|
| `AgentLoop-Simulate-System` | 设定 + 规则 + 示例 |
| `AgentLoop-Simulate-Step1` | 规则 + 示例 |
| `AgentLoop-Simulate-Step2` | 规则 |
| `AgentLoop-Simulate-Step3` | 设定 + 规则 + 示例 |
| `OutputGenerationAgent` | 设定 + 规则 + 示例 |
| `EdictRefineAgent` | 设定 + 规则 |
| `EndGameAgent` | 设定 + 规则 + 示例 |
| `TaskAgent` | 设定 + 规则 + 示例 |
| `EventManagementAgent` | 设定 + 规则 |
| `NPCUnifiedPrompt` | 设定 + 规则 |
| `CourtDiscussionPrompt` | 设定 + 规则 |
| `EasyModeContent` | 规则 |
| `db_info` | 设定 |

运行时把同一逻辑名的层文件按 设定→规则→示例 顺序**合成**为一份文本喂给模型；Mod 包内只保存原料，合成发生在服务端。Manifest 中分层条目的登记形态：键 `<逻辑名>.<层>`、路径 `prompt/<层>/<逻辑名>.md`、显式 `layer` 字段；设定层与规则层 `required: true`，示例层 `required: false`；`ScenarioMeta` 维持扁平条目（仅 `path`）。已退役的 `v2_tools` 不再注册，编辑器也不再引导编辑。

`text/default_turn_0.txt` 是首回合开场文本。提示词层文件、ScenarioMeta YAML 与 TXT 都必须是 UTF-8，单个文件受 20,000 Unicode code points 限制。信息感知规则 `content` 以 `file:` 前缀引用的推演指令长文（如 `text/gsr_yuan_kills_mao.txt`、`text/gsr_jisi_incident.txt`）是官方内置内容，由 vanilla 剧本包携带，mod 不包含这些文件。

提示词编辑器的 Markdown 预览只检查文本渲染；游戏预览不会调用真实模型，开场 TXT 也不会注入预览流程。本地导入仍使用官方服务端内容。

## 客户端资源与 `assets.json`

目录约定以 `client-assets/` 内相对路径作为官方资源键。显式映射使用根目录 `Config/assets.json`：

```json
{
  "assetItems": [
    {
      "sourceFilePath": "avatar-assets/jinyiwei.png",
      "targetFilePath": "client-assets/avatar-assets/my-avatar.webp"
    }
  ]
}
```

显式映射优先于目录约定。路径和资源族限制见[图片和资源](/resources-and-ui/images-and-assets)。

## `custom.css`

根目录 `custom.css` 可选，最多 1MB（1 MiB，即 1,048,576 个 UTF-8 字节）。编辑器保存入口（Electron 桥接与独立浏览器两条写盘路径统一生效）、游戏启动加载器和 Workshop 整目录预检都会执行这条限制（预检超限报 `custom-css` 并阻断上传）。当前没有手动设置开关；启动时锁定的各已启用 Mod 的 custom.css 会自动安全编译并注入合法规则。文件不存在、超限或编译后为空时不注入。选择器、外部 URL、保护页面与分层检查限制见[修改游戏样式](/resources-and-ui/custom-css)。
