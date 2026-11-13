---
title: 10.4 省份数据、展示资源与指令协同
description: 协调省份 CSV、地图轮廓、详情图片和省份管理指令
---

# 10.4 省份数据、展示资源与指令协同

围绕一个省份至少有四种内容，它们通过约定连接，但分别保存：

```text
provinces.csv
  ├─ ming_provinces.json 中的 province.code_name
  ├─ dynamic-assets/province-overlays/<code_name>.webp
  ├─ 土地、建筑和 troops.location_province_id
  └─ province-schema.json 定义的通用管理命令
```

## 省份业务数据

先按[7.4 省份、土地、建筑与财政](/game-content/scenario-data/provinces-and-economy)完成省份、政权、土地、建筑和财政。地图编辑器不会创建这些记录。

地图运行时按 `code_name` 合并业务数据；找不到记录时仍可能画出轮廓，但详情为空并使用无数据颜色。

## 省份详情图片

省份轮廓属性面板下方会预览：

```text
client-assets/dynamic-assets/province-overlays/<code_name>.webp
```

这张图不是地图上的纹理覆盖，而是省份详情抽屉的展示图片。运行时请求键固定为 `dynamic-assets/province-overlays/<code_name>.webp`；若物理文件使用 PNG/JPEG，需通过 `Config/assets.json` 将此完整 WebP 键显式映射到同图片资源族的文件。若 Mod 没有有效覆盖，运行时回退官方资源；官方也不存在时，页面可能显示空白或默认效果。

上传和删除图片应通过省份表的结构化图片入口或资源编辑器完成。地图属性面板当前只预览，不提供上传按钮。

## 底图

- `ming-map.*`：在地图编辑器工具栏的“地图资源”中更换。

1.6.0 官方底图已经内含旧版地名。沿用官方底图时，不要再把旧版内置 `text` 要素放进 `ming_provinces.json`，否则会叠字；自制底图仍可使用符合 GeoJSON 契约的文字要素。

## 顶点图标

地图编辑器的“地图资源”面板提供城市、驿站、军队和船只四个槽位，分别保存为固定文件名：

| 槽位 | 文件主体 |
| --- | --- |
| 城市 | `city-icon` |
| 驿站 | `station-icon` |
| 军队（通用回退） | `troop` |
| 船只动画 | `boat` |

这些是全局资源，不按要素 ID 分别配置。全局军队图标继续在地图编辑器中维护，是政权专属驻军图标的回退；`boat` 只供独立船只动画使用，不对应可创建的地图 Point。

## 政权专属驻军图标

政权表为每行提供专属驻军图标入口，输出路径固定为：

```text
client-assets/dynamic-assets/garrison-icons/<regime code_name>.webp
```

该资源只支持 WebP。删除专属图标后，游戏恢复使用全局军队图标。专属图片与 `code_name` 共同构成资源身份：修改 `code_name` 前需要先在图片面板删除专属图片；删除政权行时，确认弹窗会提示并级联删除该政权的专属驻军图标，避免遗留旧文件。

## 省份指令

`client-assets/map-data/province-schema.json` 定义右键省份管理菜单。它不属于 GeoJSON，也不会随地图保存自动改变。

命令键、值类型、默认值、模板变量和回退规则以[13.6 省份指令编辑器](/advanced/game-and-ui-config/province-commands)为唯一详细说明。本章只要求确认：

- 目标页确实加载当前指令文件；
- 指令使用的省份上下文来自有效 GeoJSON 绑定；
- Preview 页面正常不等于服务端会执行命令。

## 改名操作

修改省份 `code_name` 时，至少同步：省份表、土地、建筑、军队驻地、GeoJSON、省份图片，以及事件/决议/国策中的目标行引用。省份指令通常是通用 Schema，不按单个省份存一份，但其提示词可能人工写入名称。

mod-toolkit 的 rename 工具可辅助对账：对 `provinces` 表的 `code_name` 执行 `rename --dry-run` 会把 GeoJSON 省份要素（`features[*].properties.code_name`）等全部落点按文件分组列出。该工具只读枚举、不写盘——实际改名与逐项同步仍由作者按清单手工完成。

下一篇：[10.5 Preview、运行时回退与排错](./runtime-validation)。
