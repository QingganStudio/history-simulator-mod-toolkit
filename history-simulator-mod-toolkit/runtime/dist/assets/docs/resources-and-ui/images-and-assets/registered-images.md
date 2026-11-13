---
title: 11.2 注册图片的目录与命名规则
description: 查询国策、事件、兵种、大臣、省份和地图图片的固定路径
---

# 11.2 注册图片的目录与命名规则

结构化图片注册表是 Mod Maker 扫描、保存和 Fork 图片时的共同来源。下表路径均位于 Mod 根目录的 `client-assets/` 内。

## 路径清单

| 内容 | 目录 | 文件主体 |
| --- | --- | --- |
| 国策锁定图 | `dynamic-assets/skill-icons/` | `Skill.<数字>-lock` |
| 国策解锁图 | `dynamic-assets/skill-icons/` | `Skill.<数字>-unlock` |
| 超事件 | `dynamic-assets/events/super-events/` | `<event code_name>` |
| 常规事件 | `dynamic-assets/events/normal/` | `<event code_name>` |
| 随机、固定回合、链式事件 | `dynamic-assets/events/random/` | `<event code_name>` 点号换连字符（`event.1101` → `event-1101.webp`） |
| 兵种和船只 | `dynamic-assets/troops/` | `<code_name>` |
| 势力专属兵种/船只图 | `dynamic-assets/troops/` | `<regime code_name>_<code_name>.webp` |
| 大臣四类图片 | `dynamic-assets/memorial-cards/` | `<人物资源标识>-chat/face/card/argue` |
| 省份详情图片 | `dynamic-assets/province-overlays/` | `<province code_name>.webp` 请求键 |
| 城市图标 | `home-map-assets/icons/` | `city-icon` |
| 驿站图标 | `home-map-assets/icons/` | `station-icon` |
| 军队图标 | `home-map-assets/icons/` | `troop` |
| 战乱图标 | `home-map-assets/icons/` | `war` |
| 政权专属驻军图标 | `dynamic-assets/garrison-icons/` | `<regime code_name>.webp` |
| 阶层图标 | `dynamic-assets/social-classes/` | `<social class code_name>.webp` |
| 船只动画图标 | `home-map-assets/icons/` | `boat` |
| 地图底图 | `home-map-assets/base/` | `ming-map` |

结构化图片重新扫描当前识别 png、jpg、jpeg 和 webp。需要可靠地重新打开、Fork 和删除覆盖时，优先使用这四类格式。兵种/船只图（通用图及势力变体）、政权专属驻军图标、阶层图标和省份详情图片在游戏中按完整 `.webp` 请求键加载。按目录约定放置时用 WebP；需要保留 PNG/JPEG 物理文件时，在 `Config/assets.json` 中把完整、合法的 WebP 源键显式映射到同图片资源族的 `client-assets/...` 文件。单独放置非 WebP 兵种图且没有有效映射，运行时不会请求它。

势力专属兵种/船只图在游戏内按军队所属势力优先加载，缺失时逐级回退：`{regime code_name}_{code_name}.webp` → 通用 `{code_name}.webp` → 陆军/海军默认图。势力变体可选，只为需要定制的组合提供。`mod-scaffold --table arm_configs` / `--table ship_configs` 首次建表时会附通用 WebP 占位图；交付前换成正式图。图片中的兵种名须完整匹配 `arm_configs`/`ship_configs` 的 `code_name`，势力名须完整匹配 `regimes.code_name`；Mod 与捆绑原版的实体都会被识别，因此只改原版实体图片不必复制数据行。运行时资源键统一按小写规范化。

`mod-toolkit validate` 会对真正无法由运行时请求的图片报 `asset-reference` error：`troop-image-orphan-regime`、`troop-image-orphan-troop`、`troop-image-unrecognized`、`troop-image-unmapped-format`，以及被显式映射遮蔽的 `troop-image-shadowed`。按 issue 给出的资源键修文件名或映射。此组诊断只在工具的 `validate` 中出现，不属于 Mod Maker 保存预检或 Workshop 上传 `preflight`；交付仍要求 `validate` 零 error。

## 共享命名空间

以下内容会写入同一目录，应保持标识唯一：

- `random_events`、`fixed_turn_events`、`random_events_chain`；
- `arm_configs`、`ship_configs`；
- `ministers`、`recommend_ministers`。

事件三表已有跨表标识校验。兵种/船只和两类大臣也应在创作规范中主动避免资源主体冲突。

`dynamic-assets/troops/` 同时容纳通用图和势力变体。判定先完整匹配已知兵种 `code_name`（其中可以有多个下划线），再尝试完整匹配 `<政权 code_name>_<兵种 code_name>`；不能按下划线段数拆文件名。改政权、陆军或船只 `code_name` 前用 `rename --dry-run` 查看通用图、变体图和有效显式映射的源键落点。

## 大臣资源标识

大臣文件名不是完整 `code_name`。系统先去掉开头的 `数字_`：

```text
1001_zhangsan → zhangsan
```

然后生成：

```text
zhangsan-chat.webp
zhangsan-face.webp
zhangsan-card.webp
zhangsan-argue.webp
```

`1001_zhangsan` 与 `2001_zhangsan` 会冲突，因为去前缀后都是 `zhangsan`。

## 地图资源边界

注册表只包含主底图 `ming-map`，其他地图资源需要通过通用资源树或目录约定覆盖。

省份图片用于详情抽屉，不是地图 Polygon 的填充纹理。地图编辑器中的地图资源是全局固定文件；其中全局 `troop` 是没有政权专属图标或专属图加载失败时的通用回退，`boat` 只供独立船只动画使用，不对应 `ming_provinces.json` Point。`war` 是战乱标记：省份存在进行中战役（省份接口 `campaigns` 战斗数据非空）时替代驻军图标显示，由运行时状态驱动；覆盖它即可定制战区省份的地图观感，但地图编辑器预览不会绘制它（编辑器语境没有战役状态）。

政权专属驻军图标在政权表中编辑，文件名直接使用 `regimes.code_name`：

```text
client-assets/dynamic-assets/garrison-icons/2001_ming.webp
```

不要把图片路径写入 `regimes.csv`，也不要在地图军队点上选择政权图片。军队点绑定省份，运行时根据地块当前所属政权自动选择专属图标。

阶层图标在 `social_classes` 表中编辑，文件名直接使用 `social_classes.code_name`，游戏内阶层抽屉缺图时回退 `default.webp`：

```text
client-assets/dynamic-assets/social-classes/7001_gentry.webp
```

## 扩展名更换

结构化入口保存新扩展名时，会删除同一主体的其他扩展名。例如把 `event.webp` 改成 `event.png` 后，旧 webp 会被清理。这样运行时不会因多文件同键设计产生歧义。

通用资源编辑器替换则要求扩展名保持不变；需要换格式时应使用结构化入口，或通过显式映射把旧请求键映射到同资源族的新物理文件。

下一篇：[11.3 `client-assets` 目录约定覆盖](./convention-overrides)。
