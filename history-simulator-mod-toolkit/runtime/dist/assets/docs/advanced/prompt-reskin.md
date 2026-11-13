---
title: 三层 prompt 分工写作法（换皮范式）
description: 基于 rome/sanguo 官方换皮剧本实证的 prompt 层创作方法：rules 拷贝、setting 全量重写、example 保骨架换叙事、ScenarioMeta 增键不删
---

# 三层 prompt 分工写作法（换皮范式）

> 最近代码核对：2026-09-18；验证层级：三剧本（chongzhen/rome/sanguo）prompt 树逐文件比对。逐名参考（层组合/渲染形态/消费分级/坑位）见 [Prompt 逐名参考总览](/reference/prompts/index)。

服务端运行时把同一逻辑 prompt 的层文件按 **setting → rules → example** 顺序合成为单一文本，且**不跨剧本回退**——你的 mod 不带的层，合成结果里就真的没有。三层各司其职，换皮剧本（rome = 罗马、sanguo = 三国）的官方做法是这套分工的最佳实证。

## 一图流

| 层 | 自由度 | 换皮做法（rome/sanguo 实证） |
|---|---|---|
| **rules（规则层）** | 无需关心——服务端不落库、不使用（仅展示） | 12 个文件由 `mod-scaffold --prompt` 自动携带权威副本，内容不生效也无需改写 |
| **setting（设定层）** | 最大——世界观/人物/字段语义全归你 | 10 个文件**全量重写**（不是子集！连 DB 字段速查都换名）；可给无 setting 槽的逻辑名**开新槽**（sanguo 为 Step1 开了官方没有的 setting 层） |
| **example（示例层）** | 可选 few-shot | 6 个文件**整篇重写**：保留原版分节骨架（评分体系/章节结构/输出格式），叙事 100% 换皮 |
| **ScenarioMeta（元信息）** | 术语注入面 | **只增键不删键**：rome/sanguo 新增 start_year、number_system、display_terms 新键——改键即可换称呼/纪年/术语，无需碰锁定的模板 |

## 分步操作（mod-toolkit 工作流）

1. **查条目**：先读 [逐名参考](/reference/prompts/index) 或 `vanilla_prompt` 工具——每个逻辑名的层组合、渲染形态（jinja/static/yaml）、消费分级（5 个 fatal 名缺失即崩）与坑位提示（如 NPCUnifiedPrompt 是 LLM 前缀缓存键、db_info 要与字段别名同步）。
2. **建骨架**：`mod-scaffold <目录> --prompt --example`——ScenarioMeta + setting 骨架（带写作要点）+ rules 权威副本 + 6 个 example 槽骨架一步到位。做"改良原版"型 mod 用 `--from-vanilla` fork 全量。
3. **重写 setting**：拿 `vanilla_prompt <名> --layer setting` 的原文当参照，整文件换成你的世界观。留意渲染形态：static 条目写 `{{ }}` **不会渲染**、会原样发给模型。
4. **重写 example**：`vanilla_prompt <名> --layer example` 取原版，保骨架换叙事。
5. **ScenarioMeta 增键**：称呼/纪年/术语优先改键（edict_term、era_name、display_terms.*…），模板里早已引用；新键服务端不拒但需确认有消费方（validate 会提示 scenario-meta-unknown-key）。
6. **闭环校验**：`validate` + `preflight`——变量拼错（prompt-render-residual）、setting 缺层/空白（prompt-layer）都会报出，按 `codes --code` 的修法逐条清零。

## 两条铁律

- **rules 不用管**：脚手架已携带权威副本；服务端不落库、不使用 rules 层（仅展示），本地也不再校验其内容——想表达不同机制语义，改 setting/example。
- **example 不是必选但换皮必改**：留着原版示例 = 给模型喂崇祯叙事当 few-shot，你的世界观会被示例带偏。

## 字段口径：`_id` 与 `_code_name` 的分工

数据层与 prompt 层对同一个外键字段使用**两套名字**，改写 db_info、setting 或 example 层的数据锚定文本时必须遵守：

| 数据层真实列名（CSV / 事件条件 JSON） | prompt 层别名 | 出现表 | 引用目标 |
|---|---|---|---|
| `regime_id` | `regime_code_name` | provinces、troops | `regimes.code_name` |
| `location_province_id` | `location_province_code_name` | troops | `provinces.code_name` |
| `party_id` | `party_code_name` | ministers、recommend_ministers | `parties.code_name` |

- **prompt 层只写 `_code_name` 别名**：别名与服务端工具契约的 function call 参数同名，写对能显著提高推演命中率。原版 db_info 全文找不到一个 `_id` 列名，就是这套口径的实证。
- **反例**：把 `schema-table` 工具或 CSV 表头查到的 `regime_id` 直接抄进 db_info / setting / example——校验不会报错，但模型的 function call 命中率会静默下降。
- **数据层保持 `_id` 不变**：CSV 列名、事件条件/效果 JSON 的 `column_name`（如 random_events、game_state_rules 的 filter）、`field-definitions.json` 一律用真实列名 `regime_id` / `location_province_id` / `party_id`。

## 相关主题

- [文本、对话与 AI 提示词](/advanced/texts-and-prompts)——三层机制与登记形态
- [Prompt 逐名参考总览](/reference/prompts/index)——14 个逻辑名的机器可读契约面
- [错误码总目录](/reference/error-codes)——prompt 族 7 码的修法
