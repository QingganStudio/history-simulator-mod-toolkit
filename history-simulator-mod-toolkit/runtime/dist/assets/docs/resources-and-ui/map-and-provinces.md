---
title: 地图与省份总览
description: 组合地图底图、GeoJSON 要素、省份业务数据和交互资源
---

# 地图与省份总览

游戏地图由多条独立链路叠加：底图负责视觉背景，GeoJSON 负责轮廓、点位和纯展示文本，`provinces` 与 `troops` 提供业务详情，图标和省份图片提供界面资源，省份指令则来自另一份客户端 JSON。

## 本章专题

| 目标 | 专题 |
| --- | --- |
| 理解 1920×1080 坐标和 GeoJSON 格式 | [10.1 地图坐标系与 GeoJSON 契约](./map-and-provinces/geojson-contract) |
| 导入底图、编辑要素并正确保存 | [10.2 地图编辑器操作与保存](./map-and-provinces/editor-workflow) |
| 配置省份、城市、驿站、军队点位和文本 | [10.3 地图要素类型与业务绑定](./map-and-provinces/features-and-bindings) |
| 协调省份表、图片和指令 | [10.4 省份数据、展示资源与指令协同](./map-and-provinces/province-integration) |
| 验证运行时加载和官方回退 | [10.5 Preview、运行时回退与排错](./map-and-provinces/runtime-validation) |
| 配置视图、区间和势力颜色 | [10.6 地块视图与势力颜色](./map-and-provinces/view-colors) |

## 七类内容

| 内容 | 输出或来源 | 主要消费者 |
| --- | --- | --- |
| 底图 | `client-assets/home-map-assets/base/ming-map.*` | 地图背景图层 |
| 几何、点位和文本 | `client-assets/map-data/ming_provinces.json` | 省份、城市、驿站、军队和文本图层 |
| 地块视图与颜色 | `client-assets/map-data/province-view-config.json` | 视图菜单、势力颜色、指标区间和交互样式 |
| 地图行为 | `client-assets/map-data/map-behavior-config.json` | [13.9 地图行为](/advanced/game-and-ui-config/map-behavior-config) |
| 省份业务数据 | `provinces` 及土地、建筑、军队表 | 详情、着色、管理操作 |
| 地图和省份图片 | `home-map-assets/icons/`、`dynamic-assets/province-overlays/` | 点位图标和省份详情页 |
| 省份指令 | `client-assets/map-data/province-schema.json` | 省份右键管理菜单 |

这些文件不会互相自动生成。地图轮廓正常不代表业务省份、指令或服务端场景已经有效。

## 推荐制作顺序

1. 在 `provinces` 中建立稳定省份标识和政权归属。
2. 准备与 1920×1080 坐标对应的底图和 GeoJSON。
3. 导入地图，逐个绑定省份 `code_name`。
4. 添加城市、驿站和军队聚合点位。
5. 按需添加纯展示文本并调整字体、描边、方向和缩放方式。
6. 配置底图、图标和省份详情图片。
7. 在独立的“地块视图配置”编辑器中调整势力颜色、指标色带和公共样式。
8. 保存 Mod；关闭地图编辑器或视图配置编辑器只保留当前会话状态，不等于写盘。
9. 在 Preview 检查几何、文本、地块配色和本地 Mock 组合。
10. 在打包版检查协议和资源回退，在 Workshop 场景检查真实省份数据。

## 重要边界

- `ming_provinces.json` 只接受 `province`、`city`、`station`、`troop`、`text` 五类要素；`province` 必须是 Polygon，其余必须是 Point。
- ModMaker 打开、导入和保存预检、Electron 发布预检以及游戏运行时共用同一整图契约。任何不支持的类型（包括旧 `army` Point）都会阻断或触发官方整图回退，不做迁移或兼容。
- 船只不是 `ming_provinces.json` 要素。动画继续使用独立的 `boat_path.json` 和 `home-map-assets/icons/boat.*`，并且不接收鼠标事件。
- 命令线为带方向流动虚线的弧线，附命令类型图标与端点标记；命令热区支持悬停（高亮本组、调暗其他、展示 Tooltip）与点击（打开军队命令管理面板）。命令视觉留在 overlayPane，层级低于其上方的文本、船只、城市、驿站和驻军不变。
- `map-behavior-config.json`（地图缩放上限等地图行为覆盖）的字段规则、校验与回退边界见[13.9 地图行为](/advanced/game-and-ui-config/map-behavior-config)。

从[10.1 地图坐标系与 GeoJSON 契约](./map-and-provinces/geojson-contract)开始。下一章：[图片和资源](./images-and-assets)。
