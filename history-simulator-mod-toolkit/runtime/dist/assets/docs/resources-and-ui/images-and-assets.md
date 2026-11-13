---
title: 图片和资源总览
description: 选择正确资源入口，并理解目录覆盖、显式映射、安全限制和运行时回退
---

# 图片和资源总览

Mod 资源不是写入 CSV 的磁盘路径。游戏代码请求一个逻辑资源键，Electron 启动时建立当前 Mod 的资源索引；找到有效覆盖就读取 Mod 文件，否则回退官方资源。

## 本章专题

| 目标 | 专题 |
| --- | --- |
| 选择结构化图片入口或通用资源树 | [11.1 资源编辑器与三种工作流](./images-and-assets/editor-workflows) |
| 查找事件、人物、国策和地图图片命名 | [11.2 注册图片的目录与命名规则](./images-and-assets/registered-images) |
| 使用同路径文件覆盖官方资源 | [11.3 `client-assets` 目录约定覆盖](./images-and-assets/convention-overrides) |
| 把一个运行时键映射到不同物理文件 | [11.4 `Config/assets.json` 显式映射](./images-and-assets/explicit-mappings) |
| 理解格式、大小、像素和路径安全限制 | [11.5 格式、限制、安全检查与回退](./images-and-assets/formats-and-limits) |
| 区分 Preview、Electron dev 和打包版 | [11.6 分环境验证与故障排查](./images-and-assets/validation) |

## 资源覆盖模型

```text
游戏请求键，例如 avatar-assets/jinyiwei.png
  │
  ├─ Config/assets.json 显式映射
  ├─ client-assets/avatar-assets/jinyiwei.png 目录约定
  └─ 官方资源回退
```

显式映射在目录扫描后应用，因此同键时覆盖目录约定。

## 两类编辑入口

- **结构化图片入口**：在国策、事件、大臣、兵种、省份和地图编辑器中按实体 ID 管理，自动生成文件名并支持删除 Mod 覆盖。
- **通用资源编辑器**：浏览资源树、搜索、预览、新增媒体和同扩展名替换；当前没有删除或恢复按钮。

优先使用结构化入口管理已注册图片。通用资源树适合任意界面媒体、音频、视频、字体和原版同路径覆盖。

## 基本原则

1. 新资源统一放在 Mod 根目录的 `client-assets/` 下。
2. `client-assets/` 内部路径镜像游戏运行时请求键。
3. 不把盘符、绝对路径、`../` 或文件 URL 写进内容表。
4. 图片尽量使用注册入口支持的 png、jpg、jpeg 或 webp。
5. 保存后仍要在重启后的打包版检查资源索引和官方回退。

从[11.1 资源编辑器与三种工作流](./images-and-assets/editor-workflows)开始。下一章：[修改游戏样式](./custom-css)。
