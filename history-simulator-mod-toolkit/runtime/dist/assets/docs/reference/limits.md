---
title: 文件与格式限制
description: Mod 总量、CSV、JSON、文本、资源、路径和 Workshop 预览图限制
---

# 文件与格式限制

> 适用范围：当前内容预检契约 v2 与 Windows Electron 资源索引；最近代码核对：2026-10-14；页面类型：限制参考。

## 总体限制

| 项目 | 限制 |
|---|---:|
| Mod 目录未压缩总大小 | 1000MB |
| 未设置专用上限的注册文件 | 50MB |
| 单个 CSV 单元格 | 10,000 Unicode code points |
| 单个 JSON | 最多 1,000 条（顶层或最大嵌套数组） |
| 单个 Markdown 提示词层文件（13 个逻辑名，磁盘形态至多 28 个层文件：10 setting + 12 rules + 6 example）/ `prompt/ScenarioMeta.yaml` / `text/` 目录 TXT（1 个：`default_turn_0`） | 20,000 Unicode code points |
| `custom.css` | 1MB（1 MiB，即 1,048,576 个 UTF-8 字节） |
| 单表最多展示的预检问题 | 20 条 |
| 单个 Mod 最多展示的预检问题 | 100 条 |

问题被截断不代表其余问题已通过。先修复已显示问题，再重新检查。

这些限制并非由同一道检查执行。Mod Maker 的目录预算先计算结构化输出快照和目标目录已有文件，随后才单独写入样式草稿；Workshop 整目录预检会按实际落盘文件复核 1000MB 总量。`custom.css` 的 1MB 专项上限由编辑器保存入口（Electron 桥接与独立浏览器 File System Access API 两条写盘路径统一生效）、游戏启动加载器和 Workshop 整目录预检执行。不要用预览编译的通过替代磁盘文件的字节检查。

## CSV 行数和大小

| 上限 | 表 |
|---|---|
| 300 行 / 5MB | `regimes`、`parties`、`social_classes`、`land_tax_configs`、`equipments`、`ship_configs`、`arm_configs`、`provinces`、`finances`、`super_events`、`recommend_ministers`、`recommend_base`、`initial_tasks`、`events_prediction`、`player_attributes_init_table`；派生 `recommend` |
| 300 行 / 15MB | `buildings`、`ministers`、`troops`、`normal_events`、`random_events_chain`、`special_simulations` |
| 500 行 / 5MB | `game_state_rules` |
| 500 行 / 15MB | `decisions` |
| 500 行 / 30MB | `province_land_holdings`、`random_events`、`fixed_turn_events`、`perks_table`；立绘表 `minister_assets`（派生+额外行合并计） |

行数只计算四条 metadata 之后的数据记录。含换行的引用单元格仍是一条逻辑记录。

## 编码和语法

- CSV、JSON、Markdown、TXT、YAML 和 CSS 使用 UTF-8；UTF-8 BOM 可读，但推荐无 BOM。
- CSV 必须保留前四条 metadata，列数一致，并正确引用逗号、双引号和换行。
- JSON 必须是严格 JSON：双引号键、无注释、无尾逗号。
- Manifest 使用安全相对路径和 `/`；不允许绝对路径、盘符、空段、`.`、`..`、NUL 或重复注册。
- 输出统一使用 LF；读取支持常见 CRLF/LF。

## 运行时资源限制

| 资源族 | 扩展名 | 单文件上限 |
|---|---|---:|
| JSON | `.json` | 4MB |
| 图片 | `.png` `.jpg` `.jpeg` `.webp` `.gif` `.svg` `.ico` | 32MB |
| 字体 | `.woff` `.woff2` `.ttf` `.otf` | 32MB |
| 音频 | `.mp3` `.wav` `.flac` `.aac` `.ogg` | 64MB |
| 视频 | `.mp4` `.webm` `.ogv` | 256MB |

位图 png/jpg/jpeg/webp/gif 还必须满足宽 × 高 ≤ 64MP。超限、损坏或不支持的资源不会覆盖官方文件，而是跳过并回退原版。

## Workshop 预览图

允许 png、jpg、jpeg，最大 1MB。预览图不是 Mod 内容目录中的运行时资源；上传表单会单独读取和校验。

## 路径和文件类型

- 示例和新配置统一使用 `/`；Windows 资源显式映射可规范化 `\`，但 Manifest 本身拒绝反斜杠。
- `Config/assets.json` 的目标必须是 Mod 根目录内完整的 `client-assets/...` 普通文件。
- 符号链接和非普通文件会被拒绝；不要用链接绕过大小或根目录边界。两个 Manifest 注册项如果通过硬链接指向同一物理文件，也会被判为重复注册。
- 显式映射两端必须属于同一资源族；图片族内可换图片格式，图片不能映射到音频或 JSON。
- 路径比较按规范化小写 `/` 进行；重复 source 只保留第一条有效映射。

相关：[Mod 文件结构](./file-structure) · [错误排查](./troubleshooting)。
