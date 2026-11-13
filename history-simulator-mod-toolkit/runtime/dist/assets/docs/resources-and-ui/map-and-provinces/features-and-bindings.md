---
title: 10.3 地图要素类型与业务绑定
description: 正确使用 province、city、station、troop 和 text 五种地图类型
---

# 10.3 地图要素类型与业务绑定

`properties.type` 决定运行时由哪个图层消费要素。类型写错时，整张地图会被共享契约拒绝，不会静默忽略。

## 五种类型

| 类型 | 几何 | `code_name` 含义 | 当前主地图消费者 |
| --- | --- | --- | --- |
| `province` | Polygon | `provinces.code_name` | 省份着色、详情和右键管理 |
| `city` | Point | 无强制外键 | 城市图标和提示 |
| `station` | Point | 无强制外键 | 驿站图标和提示 |
| `troop` | Point | 省份 `code_name` | 聚合该省份的陆海军 |
| `text` | Point | 无外键，`name` 是显示内容 | 纯展示文本图层 |

属性面板仅在 `province` 和 `troop` 类型显示省份标识选择器，两者都从 `provinces` 表读取选项。

地图编辑器的“要素管理”可以集中搜索、按类型筛选并定位这些要素；右侧检查器会直接显示绑定失败、重复省份键、缺失政权和未映射颜色等问题，但最终保存仍以统一内容预检为准。

## 省份轮廓

省份 Polygon 的 `code_name` 应与业务省份一一对应。运行时会把 GeoJSON 要素与 `provinces` 中同标识记录合并为 `detail`，用于：

- 主次领地着色；
- 灾情、民心、动乱、城防和税收视图；
- 点击详情；
- 右键省份管理。

同一业务省份若拆成多个 Polygon，当前编辑器允许它们使用相同 `code_name`，但点击、缩放和交互是否符合设计需要单独测试。

## 城市和驿站

城市和驿站只使用 Point 坐标、名称和图标。运行时提示主要读取 `properties`，不会自动从 `provinces` 补充业务详情。

图标分别是：

```text
home-map-assets/icons/city-icon.*
home-map-assets/icons/station-icon.*
```

编辑器和游戏端按地图单位保持图标相对大小，缩放后会重新计算像素尺寸。图标默认半径 5 地图单位（边长 10×10）；可用要素级 `properties.iconSize`（半径语义，(0, 100]）逐点覆盖，见 10.1 属性结构。

## 军队聚合点

`troop` 点的 `code_name` 不是某支部队 ID，而是省份 ID。运行时会查找所有 `troops` 中驻扎省份相同的陆军和海军，并把结果数组放到该点位的 `detail`。

没有匹配部队时，主地图不会绘制这个军队点。点击和右键操作都是针对该省份聚合出的部队列表。

军队点的图标也不从聚合出的某支部队推断。运行时先读取该省份当前所属政权的 `code_name`，再请求：

```text
dynamic-assets/garrison-icons/<regime code_name>.webp
```

专属图标不存在、政权标识不安全或图片加载失败时，回退地图编辑器维护的全局 `home-map-assets/icons/troop.webp`。因此地块易主后会自动切换图标，不需要修改 GeoJSON 点位。军队图标默认半径自 1.6.x 起由 7 统一为 5（与城市、驿站一致）；同样可用 `properties.iconSize` 逐点覆盖。

地图编辑器使用相同的领土政权和回退顺序预览图标，但为了允许作者继续放置和修复点位，即使当前业务表没有匹配部队，作者态地图仍会显示该 Point。正式游戏仍会过滤没有实际驻军的点位。

## 船只动画不是地图要素

船只动画读取独立的 `map-data/boat_path.json` 并使用 `home-map-assets/icons/boat.*`。`boat` 是资源槽位和动画概念，不是 `ming_provinces.json` 的 `properties.type`；旧 `army` Point 已彻底移除，也不会转换成 `troop` 或船只。

## 文本类型与层级

`text` 不关联 `provinces`、`troops` 或后端实体，只负责静态展示。运行时以坐标锚定的 DOM Marker 创建 HTML 文本，DOM 内容通过 `textContent` 写入；文本中的标签和脚本只会按字面显示。

地图叠放次序从低到高为：底图与省份、命令线/命令图标/端点标记和命令热区（编辑器中同处 `overlayPane`，按添加顺序排列）、`mapTextPane`、`mapAnimationPane`（船只）、`mapCityPane`、`mapStationPane`、`mapTroopPane`，其上是共享 Tooltip、弹层和页面 UI。文本始终穿透鼠标；船只也不接收鼠标事件。实体重叠时稳定满足“驻军 > 驿站 > 城市”，不依赖 Hook 或 DOM 添加顺序。

## 检查清单

- Polygon 全部是 `province`。
- 每个省份轮廓都选择存在的业务省份。
- `troop.code_name` 使用驻军省份，而不是部队 ID。
- 政权专属驻军图标使用 `regimes.code_name` 命名，并保存为 WebP。
- 城市和驿站坐标、名称及图标正确。
- 文本内容、坐标和样式通过预检，并确认不会遮挡省份交互。
- 文件中没有旧 `army` Point 或其他未知类型。

下一篇：[10.4 省份数据、展示资源与指令协同](./province-integration)。
