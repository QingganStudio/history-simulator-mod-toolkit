---
title: 7.4 省份、土地、建筑与财政
description: 组合省份归属、土地税率、建筑和国库数据
---

# 7.4 省份、土地、建筑与财政

省份是地图、军队、建筑、土地和财政总览共同使用的业务实体。地图上出现一个省份轮廓，并不代表省份业务数据已经完整。

## 数据关系

```text
regimes ── provinces ── province_land_holdings ── land_tax_configs
                  ├──── buildings
                  ├──── troops
                  └──── GeoJSON properties.code_name

finances ── 提供国库、内帑和各类收支初值
```

## 土地类型和税率

先在 [`land_tax_configs`](/reference/tables/land_tax_configs) 中创建土地类型。这里的 `code_name` 是土地持有记录引用的稳定键，税率字段则参与 Preview 总览中的近似汇总。

修改税率会影响所有引用该土地类型的持有记录。删除土地类型前，先迁移或删除对应的 `province_land_holdings`。

## 省份

在 [`provinces`](/reference/tables/provinces) 中：

1. 分配稳定 `code_name`。
2. 使用 `regime_id` 引用 `regimes.code_name`。
3. 填写人口、民心、城防、灾情等当前 Schema 字段。
4. 保存后再到地图编辑器绑定相同 `code_name`。

`disaster_status` 和 `disaster_level` 是当前少数可选字段。可选不表示任意内容都有效；需要灾情展示时仍应使用游戏消费者认识的值。

## 省份土地

每条 [`province_land_holdings`](/reference/tables/province_land_holdings) 同时引用：

- `province_code → provinces.code_name`
- `land_type_code → land_tax_configs.code_name`

同一省份可以有多类土地。面积和税率应使用一致的数量级；Preview 会用这些数据生成税收近似值，适合发现异常倍数，但不是服务端结算公式。

## 建筑

[`buildings`](/reference/tables/buildings) 记录建筑身份、所在省份及其效果。创建前先确保目标省份存在。删除省份时，建筑不会因为地图要素消失而自动迁移。

如果建筑效果引用其他表或字段，除了外键预检，还要检查目标字段在真实服务端场景中是否可修改。

## 财政

[`finances`](/reference/tables/finances) 保存当前回合、国库、内帑及收入支出项。它与土地和建筑不是直接外键关系，但会在总览页面被组合展示。

建议先填写一个可解释的平衡状态，再逐项加入收入和支出。避免为了让 Preview 汇总“看起来正确”而反向修改真实初始值；最终结算仍以 Workshop 服务端为准。

## 与地图的边界

省份表负责业务数据；`client-assets/map-data/ming_provinces.json` 负责轮廓和点位。两者通过 `code_name` 连接。详细格式见[第 10 章：地图与省份](/resources-and-ui/map-and-provinces)。

只有表记录没有轮廓时，列表和汇总可能存在，但地图无法点击；只有轮廓没有表记录时，地图要素缺少详情并会使用无数据样式。

## Preview 组合

省份和总览 Mock 会读取：省份、政权、军队、建筑、土地持有、土地税率、财政和全局配置。检查时应同时查看：

- 省份归属和主次领地；
- 人口、民心、灾情和城防视图；
- 土地税率与面积数量级；
- 建筑、驻军和国库汇总；
- 地图轮廓与省份详情是否对应。

下一篇：[7.5 装备、兵种、船只与军队](./military-data)。
