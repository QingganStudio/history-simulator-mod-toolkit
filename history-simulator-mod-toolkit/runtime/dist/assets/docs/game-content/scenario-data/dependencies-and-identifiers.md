---
title: 7.1 数据依赖顺序与稳定标识
description: 理解基础表之间的引用方向，安全规划 code_name 和分批保存
---

# 7.1 数据依赖顺序与稳定标识

`code_name` 是跨表引用、地图绑定和图片命名共同使用的稳定标识。显示名称可以调整，已经被其他内容引用的 `code_name` 不应直接改写。

## 依赖层级

可以把制作顺序理解为以下五层：

| 层级 | 内容 | 说明 |
| --- | --- | --- |
| 基础层 | 政权、党派、阶层、土地税、装备、船只、开局输入 | 先确定可被引用的字典和基础记录 |
| 人物层 | 大臣、兵种、举荐大臣 | 引用基础层标识 |
| 省份层 | 省份、土地持有、举荐基础 | 建立主要业务实体和匹配关系 |
| 组合层 | 建筑、军队、财政 | 同时依赖政权、省份或配置表 |
| 玩法层 | 事件、决议、国策 | 在基础剧本稳定后再制作 |

层级不是文件加载的技术编号，而是安全的创作顺序。提前创建高层记录通常会得到空下拉框或保存时的外键错误。

## 关键引用

| 来源 | 目标 |
| --- | --- |
| `ministers.party_id` | `parties.code_name` |
| `provinces.regime_id` | `regimes.code_name` |
| `province_land_holdings.province_code` | `provinces.code_name` |
| `province_land_holdings.land_type_code` | `land_tax_configs.code_name` |
| `troops.regime_id` | `regimes.code_name` |
| `troops.location_province_id` | `provinces.code_name` |
| `troops.composition` 的陆军条目 | `arm_configs.code_name` |
| `troops.composition` 的海军条目 | `ship_configs.code_name` |
| `recommend_ministers.party_id` | `parties.code_name` |
| `recommend_base.code_name` | `recommend_ministers.code_name` |

`recommend_base` 选择举荐大臣后会带出只读姓名，但真正的保存匹配仍以 `code_name` 为准。

> **prompt 层的字段口径不同**：本页表格是数据层真实列名，用于 CSV 表头与事件条件 JSON 的 `column_name`。写 prompt 层文本（`prompt/setting/db_info.md`、example 层数据锚定）时，上表三个 `_id` 外键要改用 `_code_name` 别名——`regime_id`→`regime_code_name`、`location_province_id`→`location_province_code_name`、`party_id`→`party_code_name`，别名与服务端工具契约参数同名，可提高推演 function call 命中率。详见 [三层 prompt 分工写作法 · 字段口径](/advanced/prompt-reskin#字段口径id-与-code_name-的分工)。

## 规划稳定标识

建议在录入前建立一张标识清单，并遵循三条规则：

1. 标识表达身份，不表达会变化的状态，例如不要把当前官职写进大臣标识。
2. 同一资源命名空间内保持唯一；大臣资源还会去掉开头的数字前缀。
3. 地图、图片和 CSV 使用完全相同的大小写与拼写。

例如 `1001_beijing` 可以同时作为省份表主键、GeoJSON 的 `properties.code_name` 和省份详情图片的文件名主体。

## 改名和删除检查

编辑器不会把所有跨文件引用都当作一次数据库迁移自动处理。改名或删除前应：

1. 搜索全部源表和 JSON 中的旧标识。
2. 检查地图几何和注册图片文件名。
3. 检查事件、决议、国策效果中是否使用该记录。
4. 修改引用端后再修改或删除源记录。
5. 保存，重新运行内容预检和目标页面 Preview。

删除省份尤其要检查土地、建筑、军队和地图要素；删除兵种或船只要检查所有军队的 `composition`。

## 分批保存建议

- 第一批：政权、党派、阶层、税率和装备。
- 第二批：人物、举荐、兵种和船只。
- 第三批：省份、土地、建筑、财政和军队。
- 第四批：开局属性、任务、预测和特殊推演。

每批保存后先清除当前层的错误，再继续制作下一层。这样外键错误能定位到刚刚新增的少量记录。

下一篇：[7.2 政权、党派与社会阶层](./political-foundations)。
