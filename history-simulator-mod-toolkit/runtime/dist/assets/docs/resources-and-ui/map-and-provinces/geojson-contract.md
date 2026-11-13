---
title: 10.1 地图坐标系与 GeoJSON 契约
description: 理解固定地图边界、坐标顺序、要素属性和导入支持范围
---

# 10.1 地图坐标系与 GeoJSON 契约

地图使用固定的平面直角坐标系（设计空间），不是经纬度地图。设计空间原点在左下角（西南），宽 1920、高 1080（x 向东、y 向北递增），底图铺满整个设计空间。

## 坐标顺序

GeoJSON 坐标仍按 `[x, y]`，也就是代码中的 `[lng, lat]` 保存。显示时由渲染引擎内部换序，文件中始终按 `[x, y]` 书写。

```json
{
  "type": "Point",
  "coordinates": [960, 540]
}
```

这个点位于设计底图中心。不要把真实地理经纬度直接导入，也不要把 `[x,y]` 颠倒。

## 文件根结构

地图文件必须是 `FeatureCollection`：

```json
{
  "type": "FeatureCollection",
  "features": []
}
```

ModMaker 打开、导入和保存预检、Electron 发布预检以及游戏运行时都会执行同一整图契约。只要任一要素类型、几何或 Point 坐标不合法，作者工具就会拒绝该文件，运行时则回退整张官方地图。

## 支持的几何

| 几何 | 编辑器用途 | 支持情况 |
| --- | --- | --- |
| `Polygon` | 省份轮廓 | 支持 |
| `Point` | 城市、驿站、驻军聚合点和文本 | 支持 |
| `MultiPolygon` | 多块省份 | 当前编辑器不导入 |
| `LineString` | 路径 | 当前 `ming_provinces.json` 编辑器不导入 |
| 其他类型 | — | 不导入 |

岛屿或多块领地若需要一个省份表达，当前编辑器不能直接用 `MultiPolygon`。应先根据产品表现拆成受支持的 Polygon，并在实际页面验证点击和详情语义。

## Polygon 环规则

每个 Polygon 的 `coordinates` 是**环数组**，逐环满足以下规则：

- 每个环至少 4 个坐标（含首尾闭合点），且首尾坐标必须相同（闭合）；
- 多环（外环 + 洞）合法：带洞省份可用多个环表达，外环与洞都须各自闭合；
- 坐标必须是有限数字对（`NaN` / `Infinity` / 非数字拒收）；
- **多边形顶点允许轻微越出 1920×1080 设计空间**——原版沿海省实测约有 -10px 的越界出血，视觉上被地图容器裁掉；0–1920 / 0–1080 的范围约束仅是**点要素**的摆放契约；
- 自交、重叠等拓扑问题不校验（编辑器同样不查，三端口径一致）。

```json
{
  "type": "Polygon",
  "coordinates": [
    [[100, 100], [200, 100], [200, 200], [100, 200], [100, 100]],
    [[150, 150], [170, 150], [170, 170], [150, 170], [150, 150]]
  ]
}
```

环校验为 `json-schema` 码、error 级：validate 内环与 preflight 上传门禁、编辑器保存门禁与游戏运行时加载共用同一契约——任一端口发现的环问题，其余端口同样会拦或回退。

## 省份对齐检查

`province` 要素的 `code_name` 必须能在 `provinces` 表中找到对应行。失配时报 `json-fk-target-missing`（error 级，validate 与 preflight 双侧拦截），报错消息会提示「该地区上图但详情为空」——即地图画出轮廓、点击却没有业务数据。

反方向是 warning：`provinces` 表中存在某行、地图却没有对应省份要素时，报 `province-not-on-map`（`asset-reference` 码，仅 validate 报，不阻断上传）——有意留出「表里有、地图暂不画」的省份可忽略。

两个方向都**仅当 Mod 提供了地图覆盖文件**（`client-assets/map-data/ming_provinces.json`）才检查；未提供地图覆盖时走官方地图，无从对账。

## 属性结构

编辑器认识以下字段：

```json
{
  "id": "稳定的要素内部 ID",
  "name": "地图显示名称",
  "type": "province",
  "code_name": "1001_beijing",
  "iconSize": 5
}
```

`id` 用于编辑器内部选择图层；`code_name` 用于绑定业务记录。两者职责不同。导入时没有 `id` 会生成随机 UUID，保存后写回。

`properties.type` 只允许 `province`、`city`、`station`、`troop`、`text`。旧 `army` Point 和其他未知类型不会迁移、转换或静默跳过。

作者只应维护当前契约定义的 `properties`。已知字段 `id`、`name`、`type`、`code_name`、`iconSize` 以属性面板当前值为准。

### 图标大小（iconSize）

`city`、`station`、`troop` 点要素可选的图标大小覆盖，**半径语义**：图标的地图边长 = 2 × iconSize（地图单位，画幅 1920×1080）。可用属性面板的「图标大小」编辑，留空不写入该字段。

| 字段 | 合法值或范围 | 默认值 |
| --- | --- | --- |
| `iconSize` | (0, 100] 内有限数字（半径，地图单位） | 5 |

非法值（字符串、越界、NaN 等）在保存预检与上传门禁按 `json-schema` 错误拒绝；游戏运行时对未过校验的值回退默认 5。写在 `province`、`text` 要素上的合法值是惰性字段——校验放行，但游戏不消费。军队（troop）图标的默认大小自 1.6.x 起由半径 7 统一为 5。

## 文本 Point 契约

地图文本与其他要素保存在同一个 `ming_provinces.json`，不需要新增 manifest 键或资源文件。它必须是 `Point`，并使用 `properties.type = "text"`；`name` 就是显示内容，不是业务外键。

1.6.0 官方 `ming-map.webp` 底图已经画有旧版内置地名；当前内置剧本的 GeoJSON 已清除对应的 119 个旧 `text` 要素。沿用官方底图时，从旧剧本拷来的同一批文字会与底图叠加。`mod-toolkit validate` 只对身份、文字和坐标均匹配的旧要素给 `asset-reference/legacy-map-text-overlap` warning；移除这些旧要素，或提供有效的 `home-map-assets/base/ming-map.webp` 自定义底图覆盖。新的自定义 `text` 仍合法。此提醒仅在 `validate`，不属于上传 `preflight`。

```json
{
  "type": "Feature",
  "geometry": { "type": "Point", "coordinates": [960, 540] },
  "properties": {
    "id": "text-ming-title",
    "type": "text",
    "name": "大明疆域",
    "textStyle": {
      "fontFamily": "game",
      "fontSize": 16,
      "fontWeight": 500,
      "color": "#000000",
      "strokeColor": "#29180f",
      "strokeWidth": 2,
      "letterSpacing": 4,
      "lineHeight": 1.2,
      "textAlign": "center",
      "rotation": 0,
      "orientation": "horizontal",
      "scaleMode": "map",
      "opacity": 1
    }
  }
}
```

`textStyle` 只接受以下白名单值：

| 字段 | 合法值或范围 | 默认值 |
| --- | --- | --- |
| `fontFamily` | `game`、`sans`、`serif` | `game` |
| `fontSize` | 8–256 地图设计单位 | 16 |
| `fontWeight` | 300、400、500 | 400 |
| `color`、`strokeColor` | 六位或八位十六进制颜色 | `#000000`、`#29180f` |
| `strokeWidth` | 0–16 | 0 |
| `letterSpacing` | -16–64 | 0 |
| `lineHeight` | 0.5–4 倍 | 1.2 |
| `textAlign` | `left`、`center`、`right` | `center` |
| `rotation` | -180–180 度 | 0 |
| `orientation` | `horizontal`、`vertical` | `horizontal` |
| `scaleMode` | `map` 随地图缩放；`screen` 保持屏幕字号 | `map` |
| `opacity` | 0–1 | 1 |

`scaleMode = "map"` 时，字号、描边和字距都以地图设计单位解释，并按精确的 `2 ** zoom` 比例连续缩放；小数 zoom 不会取整为整级比例。`scaleMode = "screen"` 时这些尺寸保持为屏幕尺寸。文本仍是坐标锚定的 Marker HTML 文本，不是 SVG 路径；坐标投影和锚点移动由渲染引擎负责，样式尺寸由文本图层按同一 zoom 比例同步。

内容不能为空或只包含空白字符，最多 256 个 Unicode code point、4 行；坐标必须位于 0–1920 和 0–1080 范围内；单张地图最多 300 个文本要素。Mod Maker 会暂时保留尚未填写内容的文本并显示编辑器占位框，方便继续定位和编辑，但保存预检仍会拒绝该要素。任意 HTML、脚本、URL 和非白名单 CSS 都不会被解释。

## 保存规范化

保存时：

- 输出重新包装为 `FeatureCollection`；
- 坐标四舍五入到小数点后五位；
- Leaflet 图层重新转换为 GeoJSON；
- 空 `code_name` 不会写入该属性；
- 文本的 `textStyle`、点要素的 `iconSize` 和旧要素的未知扩展属性会无损保留（`iconSize` 为空时不会写入该字段）；
- 只导出当前画布中成功注册的 Point 和 Polygon。

导入外部文件前保留备份。包含未知类型、错误几何或越界 Point 的文件会在修改画布前被整体拒绝，因此不会清空当前画布，也不会只加载其中一部分；仍不要用这一行为替代版本管理。

下一篇：[10.2 地图编辑器操作与保存](./editor-workflow)。
