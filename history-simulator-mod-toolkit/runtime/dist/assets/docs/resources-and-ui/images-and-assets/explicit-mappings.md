---
title: 11.4 Config/assets.json 显式映射
description: 准确理解 source 资源键、target 物理文件和映射优先级
---

# 11.4 Config/assets.json 显式映射

显式映射把游戏请求的逻辑资源键指向 Mod 中另一份物理文件。最容易出错的是把 `sourceFilePath` 和 `targetFilePath` 写反。

## 正确方向

```json
{
  "assetItems": [
    {
      "sourceFilePath": "avatar-assets/jinyiwei.webp",
      "targetFilePath": "client-assets/avatar-assets/my-jinyiwei.png"
    }
  ]
}
```

```text
游戏请求 sourceFilePath
avatar-assets/jinyiwei.webp
              │
              ▼
Mod 物理文件 targetFilePath
client-assets/avatar-assets/my-jinyiwei.png
```

`sourceFilePath` 不带 `client-assets/`；`targetFilePath` 必须从 Mod 根目录写出完整 `client-assets/...` 路径。

## 文件位置和根结构

配置固定放在：

```text
<Mod 根目录>/Config/assets.json
```

顶层必须是对象，`assetItems` 必须是数组。每项都必须有非空字符串 `sourceFilePath` 和 `targetFilePath`；其他无效项会被过滤或在完整预检中报告。

## 扩展名和资源族

两端扩展名都必须受支持，并属于同一资源族：

- 图片可以 webp 请求映射到 png 文件；
- 音频可以 mp3 请求映射到 ogg 文件；
- 图片不能映射到音频；
- JSON 不能映射到脚本或媒体。

运行时比较资源族，而不是要求两端扩展名完全相同。

## target 的文件要求

目标路径会逐段检查：

- 位于当前 Mod 的真实 `client-assets` 内；
- 中间段都是普通目录；
- 最后一段是存在的普通文件；
- 任一段都不是符号链接；
- 真实路径解析后仍在 `client-assets` 内；
- 文件通过大小、像素和内容检查。

短路径、目录、缺失文件和越界目标都会被拒绝。

## 重复和优先级

同一个规范化 `sourceFilePath` 只能有一条有效映射。第一条有效映射生效，后续同 source 项报告重复并忽略。

索引顺序是：

1. 扫描 `client-assets` 目录约定。
2. 应用有效显式映射，覆盖同键目录结果。

## 无效映射的结果

单条映射无效时不会让整个游戏停止。该项不进入索引，页面请求会继续使用目录约定中已有结果，或回退官方资源；同时 Mod 拒绝通知和内容预检会给出路径、类型或重复原因。

下一篇：[11.5 格式、限制、安全检查与回退](./formats-and-limits)。
