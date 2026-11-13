# build-perk-tree 契约长表（国策树创作）

> 本文件是 build-perk-tree 技能的长表资料库：正文只留决策点，全表在这里。
> 内容摘自 mod 制作文档第 9 章（game-content/perk-tree/ 五篇）与内容预检契约（csv-schema 族 perk-relation-invalid 的实现侧逐条对照）。

## 1. perk-relation-invalid 子规则全表（逐条对应校验实现）

国策关系校验对 perks_table 全表执行，所有子规则共享同一 reasonCode `perk-relation-invalid`（code: csv-schema，title「国策关系无效」，rule「国策互斥和前置关系必须存在、对称且不冲突」，issue 带 `column` 与 `relatedCode` 参数）。逐条修法：

| # | 违规情形（message 形态） | 定位列 | 修法 |
|---|---|---|---|
| 1 | `X 不能以自身作为前置` | prerequisites | 从 prerequisites 移除自身 |
| 2 | `前置国策 Y 不存在` | prerequisites | 改成存在的 code_name，或移除该前置 |
| 3 | `主前置 Y 不存在` | main_prerequisite | 改成存在的 code_name，或清空主前置 |
| 4 | `主前置 Y 必须与当前国策同分类` | main_prerequisite | 主前置只连同分类节点；跨分类需求改为副前置（写进 prerequisites 即可） |
| 5 | `主前置 Y 必须包含在 prerequisites 中` | main_prerequisite | 把主前置补进 prerequisites（编辑器连线会自动做，手工改 CSV 必须自己同步） |
| 6 | `X 的前置关系形成循环` | prerequisites | 沿循环链路断开其中一条前置；画布拖线防不住手工改 CSV 造出的环 |
| 7 | `X 不能与自身互斥` | mutually_skill | 移除自斥项 |
| 8 | `互斥国策 Y 不存在` | mutually_skill | 改成存在节点或移除；两端同步处理 |
| 9 | `X 与 Y 的互斥关系必须双向保存` | mutually_skill | 两端都写：A 的 mutually_skill 含 B 且 B 的含 A |
| 10 | `X 与 Y 属于不同正式分类，互斥国策必须属于同一分类` | mutually_skill | 调整分类到同一棵树下，或撤掉该互斥——游戏端互斥索引只接受同分类关系，跨分类被静默忽略 |
| 11 | `X 与 Y 位于同一前置路径，不能互斥` | mutually_skill | 二选一：拆开前置路径，或撤掉互斥（后继要求前置已完成，前置又禁止后继完成，是逻辑矛盾） |

实现侧事实（排查时有用）：

- `Skill.0000` 是"无前置"哨兵值：出现在 prerequisites / main_prerequisite 中时不参与存在性、同分类与循环检查——不要拿它表达业务条件。
- 循环与互斥路径检查基于完整前置集（prerequisites ∪ main_prerequisite，剔除 Skill.0000）做可达性遍历，不只查直接前置。
- 同一对互斥节点的路径/分类冲突只报一次（按节点对去重）；"必须双向保存"在缺反向的一端报。

## 2. 效果四类字段分工

| 字段 | 作用 | 是否直接修改数据 |
|---|---|---|
| `current_turn_effect` | 本回合结构化数据库效果（选表、行、字段、运算和值） | 是 |
| `effect_display` | 玩家看到的效果说明 | 否 |
| `context_text` | 本回合后续推演上下文 | 否 |
| `long_term_prompt` | 后续回合长期语义 | 否 |

- 不要用展示文字替代结构化效果；也不要在提示词中要求模型再次计算已由数据库效果应用的数值（重复影响）。即时效果已扣国库时，context_text 应写明"数值影响已经应用"。
- 运算与字段类型匹配：字符串和布尔目标只能 `set`（布尔值为 `true` / `false`）；数组/对象目标用受支持的子字段。
- 目标行必须在当前剧本真实存在；数值单位和方向与 effect_display 一致；在真实场景记录执行前后值。

## 3. 13 张可写表闭集（硬边界）

`current_turn_effect` 的数据库效果只能目标下列 13 张表：

```text
arm_configs  buildings  equipments  finances  land_tax_configs  ministers  parties
provinces  regimes  ship_configs  social_classes  troops  province_land_holdings
```

`table_name` 必须写上述英文规范表名——中文名、简写、PG 别名、`_original_table_name*` 都不能授权。闭集外表（含 perks_table 自身）即 runtime-table-access-forbidden，保存和上传硬失败；旧 Mod 不会自动迁移或豁免，已导入的错误值会保留到作者修正。

## 4. 改名五处同步清单

`code_name` 是前置、互斥、图片和服务端提交共同使用的技能 ID。修改 ID 必须同步五处：

| # | 落点 | 位置 |
|---|---|---|
| ① | 其他节点的 `prerequisites` | perks_table 行内逗号分隔列表 |
| ② | 其他节点的 `main_prerequisite` | perks_table 行内标量 |
| ③ | 双向 `mutually_skill` | perks_table 互斥对两端行 |
| ④ | 两态图标 `Skill.<数字>-lock` / `Skill.<数字>-unlock` | client-assets/dynamic-assets/skill-icons/ 文件名 |
| ⑤ | 事件或决议中的 `skill_id` 条件 | 事件/决策条件 JSON 内 |

改名前先跑 `rename`（dry-run）拿按文件分组的全量落点清单——rename 的技能引用落点（skill-ref-cell）覆盖 ①②③，事件 effect/condition 内引用与 asset 约定文件名落点覆盖 ④⑤；改完用 `references` 确认旧名零残留。删除节点时同样核对：其他节点的 prerequisites / main_prerequisite / mutually_skill 清理、锁定/解锁图片是否保留、事件与决议的 skill_id 条件。

## 5. 两态图标键约定

```text
client-assets/dynamic-assets/skill-icons/Skill.<数字>-lock.<扩展名>
client-assets/dynamic-assets/skill-icons/Skill.<数字>-unlock.<扩展名>
```

- `code_name` 为 `Skill.0123` 时，编辑器内部资源 ID 使用 `0123`，最终文件仍是 `Skill.0123-lock` 与 `Skill.0123-unlock`。
- 注册图片扫描识别 png / jpg / jpeg / webp；结构化图片入口更换扩展名时会删除同一文件主体的旧扩展名（避免 .png 与 .webp 同时残留）。
- 属性面板的"画布显示：锁定/解锁"只切换编辑器预览，不修改国策状态；删除 Mod 覆盖图片后预览回退原版同路径资源，没有原版资源时显示为空。
- 一致性：锁定/解锁两图能明显区分状态、尺寸比例适合同一节点容器、ID 改名后两张图同步改名。

## 6. 布局观与 Preview 能力边界

- 画布位置保存在浏览器键 `mod-maker:perk-positions`，不进 perks_table.csv、不随 Mod 分发；自动布局只更新本地位置，不修改国策关系；拖动节点不建立/删除前置。
- 游戏端按服务端返回的节点和边重新运行 Dagre 布局，不读取编辑器本地坐标；服务端返回多条目标边时按目标去重保留首条——主前置保持唯一、明确。
- 客户端解锁检查顺序：①节点是否已解锁 → ②是否有已解锁的同分类互斥节点 → ③完整 prerequisite 列表是否全部解锁 → ④当前分类是否还有可用技能点；提交前再扫一次，阻止同时提交一对互斥国策。
- Preview 适配器会：四分类映射、按 prerequisites 生成节点和边、基本技能点与本轮选择交互、显示名称/描述/效果文字/图片。不会：输出 mutually_skill、执行 current_turn_effect、持久化真实解锁、应用长期提示词、连接真实服务端。

## 7. Workshop 验收七组用例

在测试场景中新建存档并覆盖：

1. 一个无前置根节点。
2. 一个只有主前置的节点。
3. 一个同时有主、副前置的节点。
4. 一对同分类互斥节点：两端都未解锁时各自可选；选 A 后 B 显示互斥原因且不可选；撤销本轮选择后 B 恢复可选；提交并重载后已解锁的 A 仍阻止 B。
5. 一个技能点不足时不可选择的节点。
6. 一个带数据库效果和长期提示词的节点：记录执行前后目标数据，确认即时效果只应用一次；再推进至少一个回合，确认长期提示词按预期参与推演。
7. 提交后退出并重载，检查解锁状态保持。

排错顺序：节点不显示→查分类映射、服务端场景和节点 ID；连线错误→查 main_prerequisite 与完整前置是否一致；无法解锁→按"已解锁、互斥、前置、技能点"顺序排查；Preview 可解锁但正式场景不可→重点检查互斥、服务端前置和可用技能点；编辑器位置正常但游戏排列不同→两套独立布局，检查关系而非坐标。
