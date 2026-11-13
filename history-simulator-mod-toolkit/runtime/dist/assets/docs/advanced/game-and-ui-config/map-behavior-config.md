---
title: 13.9 地图行为
description: 覆盖地图缩放上限，并理解稀疏合并、未知键整份回退与双侧校验口径
---

# 13.9 地图行为

地图行为是一个极小的覆盖片段，当前只控制一件事：玩家在游戏地图上最大能放大到整图的多少倍。它不改变底图、GeoJSON 要素或地块视图，也不会新增交互行为；本文件是「稀疏覆盖 + 运行时合并官方基底」语义，与[地块视图与势力颜色](/resources-and-ui/map-and-provinces/view-colors)同属一类。

## 输出与结构

保存输出 `client-assets/map-data/map-behavior-config.json`，作为客户端资源覆盖读取，不在 `manifest.yaml` 登记。顶层只有两个键：

| 键 | 类型 | 说明 |
| --- | --- | --- |
| `version` | number，必填 | 契约版本，目前必须为 `1` |
| `maxScale` | number，可选 | 最大放大倍数（线性；scale = 2^zoom），1–8 的有限数字；缺省继承官方值 `2` |

不存在其他合法键；扩展字段不是本文件的契约。

## 编辑步骤

1. 在地图编辑器工具栏打开「地图行为」对话框（不是侧栏独立编辑器）。
2. 调整 `maxScale`；`version` 由编辑器维护为 1，不要手工改写。
3. 保存 Mod；如需立即看效果，在 Mod Preview 中刷新——本配置支持 Preview 热更新，无需重启。

## 字段规则与最小示例

```json
{
  "version": 1,
  "maxScale": 4
}
```

- `version` 不为 `1`（缺失、非数字或别的值）会被判为不支持的版本；
- `maxScale` 若填写，必须是有限数字且落在 1–8（含端点）；写成 `3.5` 合法，写成 `"4"`、`0`、`9`、`Infinity` 不合法；
- 省略 `maxScale` 就是「不改官方缩放上限」，此时文件只剩 `version` 也合法。

## 校验与回退边界

结构校验为 `json-schema` 码、error 级：validate 内环与 preflight 上传门禁共用同一实现——不再只等上传预检，本地 `validate` 就会报出未知顶层键、版本不符或 `maxScale` 越界。

运行时按稀疏覆盖合并官方基底：

- `maxScale` 缺失 → 沿用官方值；
- `maxScale` 单值非法（越界/非有限数字）→ 警告并回退官方值；
- 整体形状非法（非对象、`version` 不支持、**未知顶层键**）→ 警告并**整份回退官方配置**，而不是忽略那一个键。

也就是说，多写一个看似无害的未来字段，会让整个覆盖片段（包括合法的 `maxScale`）一起失效。不要用「字段被静默忽略」的心态对待未知键。

## 提交前检查

- 顶层只有 `version: 1` 和可选的 `maxScale`。
- `maxScale` 在 1–8 之间且是有限数字；未把它写成字符串。
- 已理解未知顶层键会触发整份回退，没有夹带其他配置。
- 本地 `validate` 与内容预检的 `json-schema` 报错均已清零。
- 已在 Preview 中确认缩放手感（热更新即时生效）。

相关主题：[高级配置总览](../game-and-ui-config)、[地图与省份总览](/resources-and-ui/map-and-provinces)、[地块视图与势力颜色](/resources-and-ui/map-and-provinces/view-colors)、[错误排查](/reference/troubleshooting)。
