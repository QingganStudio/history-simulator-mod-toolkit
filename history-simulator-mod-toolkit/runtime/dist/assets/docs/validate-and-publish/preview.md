---
title: 预览与本地调试
description: 准确理解 Mod Maker 能读取哪些本地文件、能模拟哪些流程，以及哪些结论必须留给服务端
---

# 预览与本地调试

> 适用范围：当前 Electron Mod Maker 预览；最近代码核对：2026-10-14；验证层级：本地文件 + 前端 Mock，不连接真实游戏服务端。

Mod Maker 预览不是一组完全固定的演示数据。它会读取当前编辑器里的数据快照和 Mod 目录文件，把其中一部分转换成游戏页面需要的接口结果。因此，除了 `client-assets/`，许多 CSV 和根目录 JSON 也能在本地调试；但看到页面正确展示，不代表真实数据库、触发条件或多回合推演已经通过。

![Mod Preview 中显示的 Mock 决议页面](/screenshots/stage-4/09-mod-preview-mock.webp)

*图：预览页面使用本地快照和 Mock 数据验证界面，不代表真实服务端决议结果。*

## 数据从哪里来

预览按以下顺序取数据：

1. **编辑器内存快照**：预览窗口打开后，支持的表和 JSON 会在编辑约 300ms 后同步，尚未保存也能看到变化。
2. **Mod 目录文件**：没有内存快照时，从当前 Mod 根目录读取 CSV/JSON；保存后预览会清理旧快照并重新读取磁盘。
3. **预览兜底数据**：文件缺失、无法解析或当前功能未适配时，部分页面会显示游戏内置的预览数据或空结果。

`client-assets/` 资源使用另一条链路：`previewasset://` 优先读取当前 Mod，找不到时回退游戏原版。图片、音频、视频和字体支持本地读取；图片和样式修改还可局部热更新。不要仅凭“页面没有报错”判断已经使用 Mod 数据，最好填入一个容易识别的测试名称或数值再核对。

## 23 张可参与页面 Mock 的源表

当前 27 张可编辑源表中，有 24 张进入编辑器预览快照白名单，其中 23 张会被预览服务适配器直接消费，剩余 1 张是下面单列的 `random_events_chain`。派生 logical name `recommend` 也可进入快照，但它不是可编辑源表，不计入本节的 23 张源表。

| 能力 | 涉及表 | 预览能检查什么 |
|---|---|---|
| 基础列表与详情 | `ministers`、`parties`、`social_classes`、`equipments` | 名称、字段展示、党派关联、大臣官职与资源路径 |
| 政权、省份与总览 | `regimes`、`provinces`、`buildings`、`province_land_holdings`、`land_tax_configs`、`finances` | 主政权、省份归属、人口民心、建筑，以及按预览公式汇总的税收和财政 |
| 军队与编制 | `troops`、`arm_configs`、`ship_configs` | 陆海军分组、驻地、政权、编制、装备需求、人数/船数和维护费近似值 |
| 开局页面 | `events_prediction`、`initial_tasks`、`player_attributes_init_table` | 事件预测、任务列表/详情，以及军事、管理、政治、学识四项可识别的初始属性 |
| 事件展示 | `normal_events`、`random_events`、`super_events` | 常规/随机事件的标题、分类和选项展示；从有效超事件中随机展示一条 |
| 决议与国策 | `decisions`、`perks_table` | 决议列表、排序、执行结果展示、一次性完成状态；国策节点、前置边和基本解锁交互 |
| 举荐流程 | `recommend_base`、`recommend_ministers` | 完成一轮模拟并提交国策后，从 `code_name` 能匹配的候选中模拟一次举荐 |

这些表并不是按后端原算法运行。预览会做必要的字段转换、跨表关联和部分近似计算，让真实游戏组件能够渲染。例如财政总览会根据省份、土地、税率、军队和财政表重新计算展示值；这适合发现引用、数量级和页面布局问题，但不能作为服务端公式的权威结果。

## 4 张能力受限或未接入的源表

| 表 | 当前预览能力 | 应该怎么验证 |
|---|---|---|
| `random_events_chain` | 会进入编辑器快照，也能从磁盘解析，但当前没有游戏页面服务适配器消费它 | 在专用编辑器检查节点和连线，通过内容预检；真实链式推进等待服务端场景测试 |
| `fixed_turn_events` | 不进入预览快照，也没有预览服务适配器 | 在编辑器和内容预检检查结构；真实回合触发等待服务端场景测试 |
| `special_simulations` | 不进入预览快照，也没有预览服务适配器 | 在编辑器和内容预检检查结构；真实特殊推演等待服务端场景测试 |
| `game_state_rules` | 不进入预览快照，也没有预览服务适配器 | 在编辑器和内容预检检查结构；真实信息感知注入等待服务端场景测试 |

因此，“27 张表都能在游戏预览中调试”是不准确的。每张表的具体级别也会显示在[数据表参考](/reference/tables/)中。

## 根目录 JSON 与其他非客户端文件

| 文件 | 当前预览能力 |
|---|---|
| `global/minister_positions.json` | 支持内存热更新，用于官职分组、名称和已占用官职的页面展示 |
| `global_config.json` | 权威编辑/导入文件；支持内存热更新，参与主政权选择，并刷新预览中的全局配置消费者。Preview 不改读 `client-assets/global_config.json` 镜像 |
| `pg-table-mapping.json` | 不参与游戏页面 Mock；只做编辑、保存与内容预检 |
| `prompt/{setting,rules,example}/*.md`、`prompt/ScenarioMeta.yaml` | 只能使用提示词编辑器的 Markdown 和变量检查；游戏预览不会调用真实模型 |
| `text/*.txt` | 只做文本编辑和内容预检；当前不会注入游戏预览流程 |
| `manifest.yaml` | 用于目录身份、注册与保存检查，不是页面内容 Mock |

保存生成的 `global/recommend.csv` 是举荐流程使用的派生数据；作者应维护 `recommend_base` 和 `recommend_ministers` 两张源表。`global/minister_assets.csv` 由两表派生只读行与立绘表编辑器的额外行合并生成，在立绘表编辑器中维护，图片文件本身不参与它的生成。

## 可以模拟到什么程度

预览支持一些短流程，但行为是专门为界面检查编写的模拟：

- 决议默认可见、可执行；一次性决议会记录当前预览窗口内的执行次数，但可见条件和执行条件不会按真实数据库求值。
- 执行决议会展示 `database_effects` 和关联超事件，但不会把效果持续写回其他页面的数据状态。
- 国策可以展示节点和前置连线并执行基本解锁动作，但不会执行真实条件、效果、长期影响或服务端推演。
- 常规事件按所选分类截取少量记录；随机事件只取有限样例。条件、概率、后续链和真实效果不会完整执行。
- 超事件从有效记录中随机取一条用于展示，不判断真实触发条件和概率。
- 举荐只在“完成一轮模拟并提交国策”的预览信号之后触发一次，并从能够按 `code_name` 匹配的数据中随机选择。
- 缺失或不支持的接口可能返回固定对象、空数组或成功占位结果，使页面可以继续操作。

这些能力足以调试“文件有没有被读到、字段能否转换、关联能否展示、页面是否可操作”，不适合调试“服务端最终会不会这样判定”。

## 客户端配置、资源与样式

以下客户端 JSON 可以进入预览快照并热更新：

- `client-assets/game-config.json`
- `client-assets/key-events.json`
- `client-assets/minister-chat-prompt-presets.json`
- `client-assets/court-chat-quick-messages.json`
- `client-assets/page-operation-guides.json`
- `client-assets/map-data/ming_provinces.json`
- `client-assets/map-data/province-view-config.json`
- `client-assets/map-data/map-behavior-config.json`
- `client-assets/field-definitions.json`

保存还会生成 `client-assets/global_config.json`，供正式客户端的 `modasset://` 资源索引读取；Preview 的全局配置快照仍来自根目录权威文件。历法则相反：Preview 与 Mod Maker 都以 `client-assets/game-config.json` 为权威输入，`global/game_config.json` 只作为 Manifest/服务端副本。

`client-assets/map-data/province-schema.json` 不在编辑器快照白名单中。预览页面可以通过资源协议读取已经保存的文件，但修改后需要保存并重新加载相关页面，不能期待未保存热更新。

普通图片、音频、视频和字体从当前 Mod 的 `client-assets/` 读取并回退原版。根目录 `custom.css` 的草稿会先经过与游戏相同的选择器安全编译，再作为 `<style>` 注入预览；预览入口有自己的长度保护，不能用预览成功替代保存和启动时的 1MB UTF-8 字节检查。部分只通过磁盘资源协议加载的文件需要先保存或重新加载预览，不能把所有客户端文件都理解为未保存即热更新。

## 预览明确不能证明什么

预览会替换游戏服务方法、屏蔽真实外部请求，并把 Electron/Workshop 能力改为空实现。因此它不能证明：

- 服务端数据库成功导入了 CSV、JSON、提示词或文本；
- 事件、决议、国策、任务和特殊推演按真实条件触发；
- 数据效果真正落库，存档、恢复和多回合状态保持正确；
- AI 提示词、NPC 对话、诏书、推演报告和模型工具调用符合预期；
- Steam 上传、审核、下载、场景标签和 `X-Scenario-Id` 链路正常；
- 本地导入后服务端会读取本机 CSV；事实上，本地 `local_` Mod 仍使用官方服务端场景；
- 打包版的资源协议、ASAR 回退和 Windows 路径完全正确。

## 用 CDP 调试预览窗口

预览窗口打开时，游戏会在本机 `127.0.0.1:9333` 启动一个只绑定预览页面的 CDP 调试端点：访问 `http://127.0.0.1:9333/json/version` 与 `/json/list`（取 `webSocketDebuggerUrl`）可发现调试目标，也可经 `ws://…/devtools/page/preview` 直连预览页面。通过它可以用 `Runtime.evaluate` 检查渲染结果、computed style 与控制台告警；外部工具改写 `custom.css` 后经它推送样式热更新的完整示例见[修改游戏样式](/resources-and-ui/custom-css)。该端点只服务预览窗口，不能连接游戏主窗口。

## 推荐调试顺序

1. 先保存并通过内容预检，排除格式、必填、主键和外键问题。
2. 打开预览，用明显测试值确认页面确实读取了 Mod 数据。
3. 修改支持热更新的表或 JSON，检查相关页面是否刷新。
4. 对照本页确认该功能属于展示、派生还是模拟交互，不对 Mock 结果作过度推断。
5. 本地导入打包版，验证 `client-assets/`、`custom.css`、资源协议、路径和官方回退。
6. 服务端场景完成拉取、校验与审核后，再验证数据库内容、真实规则、AI 和多回合玩法。

下一步：[本地导入和客户端测试](./local-testing)。
