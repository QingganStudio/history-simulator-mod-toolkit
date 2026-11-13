---
title: 7.5 装备、兵种、船只与军队
description: 建立装备需求、陆海军配置和实际军队编制
---

# 7.5 装备、兵种、船只与军队

军队数据分为“可用配置”和“实际部队”两层：`equipments`、`arm_configs`、`ship_configs` 定义可组成什么，`troops` 定义开局实际有哪些部队。

## 制作顺序

1. 在 `equipments` 建立装备字典。
2. 在 `arm_configs` 建立陆军兵种及装备需求。
3. 在 `ship_configs` 建立船只配置。
4. 在 `regimes` 和 `provinces` 完成部队归属与驻地。
5. 最后在 `troops` 创建实际军队并填写编制。

字段明细见 [`equipments`](/reference/tables/equipments)、[`arm_configs`](/reference/tables/arm_configs)、[`ship_configs`](/reference/tables/ship_configs) 和 [`troops`](/reference/tables/troops)。

## 装备与兵种

装备记录提供稳定标识、名称及当前 Schema 要求的成本或属性。兵种的装备需求是结构化数据，应从已有装备中选择并填写数量，不要把整段 JSON 当作普通说明文字。

删除装备前，检查所有兵种需求。即使某装备当前没有实际库存，它仍可能被兵种配置或事件效果引用。

## 船只配置

船只和陆军兵种在实际部队编制中使用相似的引用方式，但拥有不同字段。`ship_configs.crew_count` 当前可选；缺失时不要自行补一个未经设计的默认船员数。

`arm_configs` 与 `ship_configs` 共用 `dynamic-assets/troops/` 图片命名空间。为避免图片冲突，两张表的 `code_name` 应保持全局唯一。

## 实际军队

每条 `troops` 至少需要回答：

- 属于哪个政权：`regime_id → regimes.code_name`；
- 驻扎在哪个省份：`location_province_id → provinces.code_name`；
- 是陆军还是海军；
- `composition` 中由哪些兵种或船只组成；
- 当前士气、忠诚、补给等状态是什么。

`composition` 中陆军条目引用 `arm_configs.code_name`，海军条目引用 `ship_configs.code_name`。类型与目标表不一致时，即使字符串存在，也不能得到正确的强度或船只汇总。

## 兵种与船只图片

兵种和船只图片通过结构化图片入口保存到：

```text
client-assets/dynamic-assets/troops/<code_name>.<扩展名>
```

CSV 只使用对应数据表参考中列出的字段，不填写磁盘路径或资源 URL。资源命名、扩展名和回退规则见[第 11 章：图片和资源](/resources-and-ui/images-and-assets)。

## 删除和改名

- 改兵种或船只标识：同步修改所有 `troops.composition` 和对应图片。
- 删除省份：先迁移驻扎部队和地图军队点位。
- 删除政权：先迁移其全部军队。
- 改部队标识：检查事件、决议、国策效果是否引用该行。

## Preview 与真实验证

Preview 会组合部队、兵种、船只、装备、政权和省份，计算总兵力或总船数并展示驻地关系。它适合检查编制解析和图片，但不执行真实维护费、战斗、补给或回合损耗。

Workshop 验收至少覆盖一支陆军和一支海军：检查初始数量、驻地、装备需求、页面图片，以及一回合后的维护和状态变化。

下一篇：[7.6 开局属性、任务、预测与特殊推演](./opening-data)。
