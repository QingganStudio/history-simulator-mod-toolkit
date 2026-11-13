# create-events 契约长表（事件与决议创作）

> 本文件是 create-events 技能的长表资料库：正文只留决策点，全表在这里。
> 内容摘自 mod 制作文档第 8 章（game-content/events-and-decisions/ 六篇）与内容预检契约（csv-schema 族）。

## 1. 六类事件选型矩阵

| 需求 | 使用表 | 该表的最小职责 |
|---|---|---|
| 保存一条常规历史或推演事件 | `normal_events` | 名称、描述、效果文字和分类；没有三个选项和结构化触发条件，不应拿来替代随机事件 |
| 满足条件后参与随机抽取 | `random_events` | 标题、描述、条件 + 三组选项（内容/展示/效果） |
| 在指定回合触发 | `fixed_turn_events` | 同随机事件列集合；turn 条件只能 `equal` |
| 选项会进入下一节点 | `random_events_chain` | 同随机事件列集合；后继关系保存在选项效果的 `follow_up` |
| 以独立大画面展示重大事件 | `super_events` | 标识、标题、描述和条件；图片不在 CSV，按 code_name 放资源目录 |
| 让玩家主动执行并记录次数 | `decisions` | 可见条件、执行条件、重复规则、效果、提示词和可选超事件快照的组合 |

反模式：把固定回合事件做成普通随机事件，或把连续剧情拆成互不关联的随机事件——后续验证都会变困难。

## 2. 条件类型全表（8 类）

| 类型 | 用途 | 关键输入 |
|---|---|---|
| `turn` | 比较当前回合 | 判断方式、目标回合 |
| `str_judge` | 字符串字段与固定值比较 | 表、行、字段、值 |
| `num_judge` | 数值字段与固定值比较 | 表、行、字段、运算和值 |
| `num_compare` | 两个数值字段互相比较 | 两组表/行/字段和运算 |
| `str_compare` | 两个字符串字段互相比较 | 两组表/行/字段和运算 |
| `skill` | 判断国策/技能状态 | `skill_id` |
| `decision_executed` | 判断某决议是否执行 | `decision_id` |
| `super_event_triggered` | 判断某超事件是否触发 | `event_id` |

判断方式闭集：数值与回合条件可用 `more` / `more_equal` / `minus` / `minus_equal` / `equal` / `not_equal`；字符串条件只适合 `equal` 和 `not_equal`。

其他结构事实：

- 表/行/字段保存的是技术标识，显示名称只是选择辅助——目标行改名或删除后，条件不会自动变成另一条记录。
- `finances` 是单行表：单目标条件省略 `row_name`，比较条件两侧分别省略 `row_name_1` / `row_name_2`；空字符串、`null`、`"Value"` 等占位值都不合法。
- 固定回合事件的 turn 条件恰为 `{ "type": "turn", "judge_type": "equal", "value": 整数 }`——大于/小于会破坏"固定回合"的确定性。
- 决议条件树当前只表达 `all`（所有叶子条件同时成立）；保存时移除旧 `failure_code` / `failure_text`，面向玩家的条件说明写 `condition_display`。
- 决议的 `visibility_conditions`（不满足不显示）与 `execution_conditions`（不满足可显示不可执行）分工不同，不要机械复制。

## 3. 选项三段式与效果类型

随机、固定回合和链式随机事件最多三组选项：

| 选项 | 内容（玩家看到的文字） | 展示（结果说明） | 效果（机器执行） |
|---|---|---|---|
| 1 | `c1_content` | `c1_display` | `cleaned_c1_effect` |
| 2 | `c2_content` | `c2_display` | `cleaned_c2_effect` |
| 3 | `c3_content` | `c3_display` | `cleaned_c3_effect` |

空选项组可以不配置；已填写的组必须三段齐填——只填一部分会产生"看起来有结果但数据未改变"或"数据改变但玩家不知道"。

三种效果类型：

| 类型 | 作用 | 要点 |
|---|---|---|
| `database` | 改数据表的表/行/字段 | 运算 `add` / `subtract` / `multiply` / `divide` / `set`；字符串与布尔目标只能 `set`；`divide` 的值不能为 0；指向 `finances` 必须省略 `row_name` |
| `attribute` | 改玩家属性（`military` / `management` / `politics` / `knowledge` 等） | 仅普通事件选项提供；决议效果编辑器不提供该类型，决议用 database 效果 |
| `follow_up` | 指定链式后继 | 仅 `random_events_chain`；是某个选项效果数组中的一项，每选项最多一个；事件链画布的边由这些效果反向推导 |

数组/对象字段通过 `path` 指向内部成员——不要把整对象覆盖当作普通数值加减；目标结构不支持所选操作时，改用专用子字段编辑。

## 4. 13 张可写表闭集（硬边界）

事件和决议的结构化条件读取、数据库效果写入，只能目标下列 13 张表：

```text
arm_configs  buildings  equipments  finances  land_tax_configs  ministers  parties
provinces  regimes  ship_configs  social_classes  troops  province_land_holdings
```

`table_name` / `table_name_1` / `table_name_2` 必须写上述英文规范表名——中文名、简写、PG 别名、`_original_table_name*` 元数据都不能代替。闭集外表（含事件表自身与 perks_table）使保存和上传硬失败；旧 Mod 不会自动迁移或豁免。

## 5. 六个事件专属错误码修法详表（csv-schema 族）

| reasonCode | 规则 | 修法 |
|---|---|---|
| `event-key-duplicate` | 共享事件命名空间中的非空 code_name 必须跨表唯一（random_events / fixed_turn_events / random_events_chain） | 重命名其一；随后用 `references` 追旧名的全部引用：链式选项 follow_up、决议条件的 decision_id / event_id、超事件快照 event_id、图片文件名（点号→连字符换算后的形态）、提示词与展示文字中的人工引用 |
| `fixed-turn-event-invalid` | 固定回合事件必须包含可建立索引的顶层 turn 条件 | cleaned_condition 首个条件改为 `{ "type": "turn", "judge_type": "equal", "value": 整数 }` |
| `super-event-snapshot-invalid` | 超事件快照行必须恰为 code_name/title/description/cleaned_condition 四个字段，不得包含多余字段 | 删除快照行里的多余字段；内容过时则与源 super_events 记录重新同步 |
| `decision-structure-invalid` | 决议字段必须满足共享决议结构规则 | 按 issue 的 column 定位：repeat_policy 按 schema_table 子契约选择 once / once_per_turn / cooldown / repeatable，并填写适用的正整数参数；条件树只表达 `all`；条件条目不带旧 failure_code / failure_text |
| `runtime-table-access-forbidden` | 条件读取和效果写入只能使用共享运行时表白名单中的规范表名 | 把 table_name（含 _1/_2）换成 13 张闭集表的英文规范名（见 §4）；闭集外表的需求改设计，不要绕 |
| `forbidden-singleton-row-selector` | 精确指向 finances 表时必须省略对应的 row_name 字段 | 整字段删除 row_name / row_name_1 / row_name_2——不是置空；空串、null、"Value" 同样被拒 |

## 6. 事件图资源键

| 事件类 | 请求键 |
|---|---|
| 常规事件 | `dynamic-assets/events/normal/<code_name>.webp`（code_name 原样，含点号） |
| 随机 / 固定回合 / 链式 | `dynamic-assets/events/random/<code_name 点号换连字符>.webp`（`random.tax_shortage.001` → `random-tax_shortage-001.webp`） |
| 超事件 | `dynamic-assets/events/super-events/<code_name>.<扩展名>` |

随机三表共用图片目录且点号拉平：`random.tax.001` 与 `random-tax-001` 解析到同一张图——命名时连字符形态也视为已占用。

## 7. Preview 能力矩阵与 Workshop 测试用例

| 内容 | 专用编辑器/预检 | Preview | Workshop 场景 |
|---|---|---|---|
| `normal_events` | 字段和结构 | 有限分类样例 | 真实流程 |
| `random_events` | 条件、选项、效果 | 有限样例，不算真实概率 | 条件、概率、效果 |
| `fixed_turn_events` | 固定回合专项规则 | 不进入 Preview 快照 | 指定回合触发 |
| `random_events_chain` | 画布关系、结构 | 进入快照但无页面适配器 | 分支和后继 |
| `super_events` | 记录和图片 | 有限随机展示 | 真实条件和去重 |
| `decisions` | 条件、效果、快照 | 排序和有限执行状态 | 可见/执行条件、落库和重复规则 |

Preview 适合检查：标题、描述、分类和排序；选项数量、展示文字和图片；决议一次性完成状态；页面能否解析当前结构。不要据此确认：条件求值、抽取概率、数据库持久化、回合触发、链式后继、长期提示词或存档恢复。

Workshop 测试至少准备七组用例：

1. 条件满足与不满足的两组局面。
2. 固定回合前一回合、目标回合和后一回合。
3. 链式事件每个分支及终止节点。
4. 决议执行前后目标字段对照。
5. 验证四策略、每回合与总次数上限、冷却边界；Preview可推进回合、同ID热改保留历史、手动重置。正式游戏还需验证资源效果、幂等、读档与关闭重开恢复。
6. 执行后超事件标题、描述和图片与快照一致。
7. 保存、退出并重载后，执行次数和长期状态保持正确。

排错顺序：完全不显示→查事件类型、可见/触发条件和稳定标识；有按钮不能执行→查执行条件和目标记录；显示正确但数据不变→查效果类型、表/行/字段和运算；链路中断→查当前选项效果的 follow_up；超事件内容过时→重新同步决议快照；Preview 正常但正式场景失败→以服务端条件、目标数据和场景版本为下一层证据。
