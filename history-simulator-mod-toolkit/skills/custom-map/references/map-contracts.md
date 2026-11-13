# custom-map 契约长表（自定义地图制作）

> 本文件是 custom-map 技能的长表资料库：正文只留决策点，全表在这里。
> 内容摘自 mod 制作文档第 10 章（resources-and-ui/map-and-provinces/ 六篇）与 mod-toolkit 校验实现（legacyMapText、rename 落点、地图共享契约）。

## 1. GeoJSON 契约细表

**根结构**：必须是 `FeatureCollection`（`{ "type": "FeatureCollection", "features": [...] }`）。ModMaker 打开/导入/保存预检、Electron 发布预检与游戏运行时执行同一整图契约——任一要素类型、几何或 Point 坐标不合法，作者工具拒绝该文件，运行时回退整张官方地图。

**坐标系**：Leaflet `CRS.Simple`，设计空间宽 1920、高 1080，底图铺 `[[0,0],[1080,1920]]`。GeoJSON 坐标按 `[x, y]`（即 `[lng, lat]`）存储；Leaflet 显示时转换为 `[lat, lng]`。不要把真实地理经纬度直接导入，也不要把 `[x,y]` 颠倒。底图总是拉伸到 1920×1080——图片比例不同会形变，更换底图不会自动缩放现有几何。

**几何支持**：

| 几何 | 编辑器用途 | 支持情况 |
|---|---|---|
| `Polygon` | 省份轮廓 | 支持 |
| `Point` | 城市、驿站、驻军聚合点和文本 | 支持 |
| `MultiPolygon` | 多块省份 | 当前编辑器不导入（先拆成多个 Polygon，可用相同 code_name，点击/缩放/交互单独测试） |
| `LineString` | 路径 | 当前 `ming_provinces.json` 编辑器不导入 |
| 其他类型 | — | 不导入（旧 `army` Point 不迁移、不转换、不静默跳过） |

**Polygon 环规则**（环校验为 `json-schema` 码、error 级，validate/preflight/编辑器/运行时共用同一契约）：

- 每个环至少 4 个坐标（含首尾闭合点），首尾坐标必须相同；
- 多环（外环 + 洞）合法，外环与洞都须各自闭合；
- 坐标必须是有限数字对（NaN / Infinity / 非数字拒收）；
- 多边形顶点允许轻微越出 1920×1080（原版沿海省实测约 -10 出血，视觉被容器裁掉）；0–1920 / 0–1080 的范围约束仅是**点要素**的摆放契约；
- 自交、重叠等拓扑问题不校验（编辑器同样不查，三端口径一致）；
- 单地块最多 200 个存储点（含末尾重复的闭合点）。

**属性结构**：`id`（稳定的要素内部 ID，编辑器选层用；导入缺失时生成随机 UUID 并写回）、`name`（地图显示名称）、`type`（五要素之一）、`code_name`（绑定业务记录）、`iconSize`（可选，仅 city/station/troop 点要素消费：图标半径，地图单位，边长 = 2×iconSize，(0, 100]，缺省 5——troop 默认 1.6.x 起由 7 统一为 5；非法值报 json-schema error，province/text 上的合法值为惰性字段）。只应维护当前契约定义的属性；空 `code_name` 与空 `iconSize` 保存时不写入该属性。

## 2. 五要素与业务绑定表

| 类型 | 几何 | `code_name` 含义 | 运行时消费者 |
|---|---|---|---|
| `province` | Polygon | `provinces.code_name` | 省份着色、点击详情、右键管理（要素与省份记录按 code_name 合并为 detail） |
| `city` | Point | 无强制外键 | 城市图标和提示 |
| `station` | Point | 无强制外键 | 驿站图标和提示 |
| `troop` | Point | **省份** code_name（不是部队 ID） | 聚合该省份的陆海军；无匹配部队时主地图不绘制该点 |
| `text` | Point | 无外键，`name` 是显示内容 | 纯展示文本图层（DOM 经 textContent 写入，标签/脚本只按字面显示） |

Pane 层级从低到高：原 overlayPane（底图、省份、命令层）→ mapTextPane（文本）→ mapAnimationPane（船只）→ mapCityPane → mapStationPane → mapTroopPane。文本始终穿透鼠标，不挡省份交互；实体重叠时稳定"驻军 > 驿站 > 城市"。船只动画读独立的 `map-data/boat_path.json` 与 `home-map-assets/icons/boat.*`，不是五要素契约的一部分。

## 3. textStyle 白名单全表与 text 要素上限

| 字段 | 合法值或范围 | 默认值 |
|---|---|---|
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

text 要素上限：内容不能为空或纯空白；最多 256 个 Unicode code point、4 行；坐标必须位于 0–1920 / 0–1080；单张地图最多 300 个文本要素。任意 HTML、脚本、URL 和非白名单 CSS 都不会被解释。`scaleMode = "map"` 时按 `2 ** zoom` 精确比例连续缩放（小数 zoom 不取整）。

## 4. 保存规范化规则

- 输出重新包装为 `FeatureCollection`；
- 坐标四舍五入到小数点后五位；
- Leaflet 图层重新转换为 GeoJSON；
- 空 `code_name` 不写入该属性；
- 文本的 `textStyle`、点要素的 `iconSize` 和旧要素的未知扩展属性无损保留；
- 只导出当前画布中成功注册的 Point 和 Polygon；
- 导入不是"合并"：含未知类型、错误几何或越界 Point 的文件在修改画布前被整体拒绝（不会清空当前画布，也不会只加载一部分）——仍不要用这一行为替代版本管理。

## 5. code_name 落点对账清单表（provinces 改名）

对 `provinces` 表 code_name 跑 `rename --dry-run`，落点按文件分组返回（只读枚举，不写盘）：

| # | 落点 | 位置 |
|---|---|---|
| ① | 定义行 | `pg/provinces.csv` 的 code_name 主键 |
| ② | FK 引用单元格 | 土地（province_land_holdings）、建筑（buildings）、驻军（troops.location_province_id）等 schema 声明的引用列 |
| ③ | 事件/决议/国策内联引用 | 事件 effect/condition 内的 row_name 与经 FK 列的 value、decisions.super_event 快照 |
| ④ | GeoJSON 省份要素 | `client-assets/map-data/ming_provinces.json` 的 `features[*].properties.code_name`（type=province） |
| ⑤ | 省份详情图资源键 | `dynamic-assets/province-overlays/<code_name>.webp`（含 Config/assets.json 显式映射条目） |

改完用 `references <目录> provinces <旧名>` 确认旧名零残留。stale 要素 = 上图但详情为空（validate 报 json-fk-target-missing）。省份指令（province-schema.json）是通用 Schema，不按单个省份存一份，但其提示词可能人工写入名称——改名后人工核对。

## 6. 图标键与回退链表

| 资源 | 运行时请求键（client-assets/ 下，键不带前缀） | 回退 |
|---|---|---|
| 底图 | `home-map-assets/base/ming-map.<扩展名>` | 未覆盖走官方 |
| 城市图标 | `home-map-assets/icons/city-icon.*` | 全局槽位，不按要素 ID 分 |
| 驿站图标 | `home-map-assets/icons/station-icon.*` | 同上 |
| 驻军图标（政权专属） | `dynamic-assets/garrison-icons/<regime code_name>.webp`（**仅 WebP**） | 缺失/政权标识不安全/加载失败 → 全局 `home-map-assets/icons/troop.webp` |
| 驻军图标（全局回退） | `home-map-assets/icons/troop.webp` | 政权专属图标的最终回退；地块易主自动切换，不用改 GeoJSON |
| 省份详情图 | `dynamic-assets/province-overlays/<code_name>.webp`（PNG/JPEG 物理文件须经 Config/assets.json 显式映射到该 WebP 键） | 未覆盖回退官方；官方也没有时页面可能空白 |
| 势力配色 | `map-data/province-view-config.json` 标准视图 `rule.colors`（键 = 当前 Mod `regimes.code_name`，整表替换不混官方 key） | 政权有效但未映射 → 视图 `fallbackColor` → `noDataColor`；未绑定/无有效政权 → `noDataColor` |

Mod Maker 保存时会按当前 `regimes.csv` 自动对账标准视图颜色（新增势力稳定生成、删除/改名势力清理遗留 key）。政权专属驻军图标改名前需先在图片面板删除专属图；删除政权行会级联删除其专属驻军图标。

## 7. 三个地图对账/提醒码性质对照

| reasonCode | code | severity | 方向 | 是否拦上传 |
|---|---|---|---|---|
| `json-fk-target-missing` | json-schema | error | 要素 → 表（GeoJSON 省份要素 code_name 在 provinces 表无对应行） | 是（validate 与 preflight 双侧拦截） |
| `province-not-on-map` | asset-reference | warning | 表 → 图（provinces 行无对应省份要素） | 否（仅 validate；有意留空可忽略） |
| `legacy-map-text-overlap` | asset-reference | warning | 旧地名提醒（身份/文字/坐标均匹配 119 个旧要素才触发） | 否（**validate-only，不在 47 个预检契约码内**；叠字是视觉事故，必须清零） |

三个方向都**仅当 Mod 提供了地图覆盖文件**（`client-assets/map-data/ming_provinces.json`）才检查；未提供时走官方地图，无从对账。legacy-map-text-overlap 的两个短路：提供有效 `home-map-assets/base/ming-map.webp` 自定义底图后不再提醒；地图契约失败（整图回退官方）时旧 text 实际不叠字，同样不提醒。

## 8. Preview 能力矩阵与打包版检查清单

Preview 可查：底图/边界/点位对齐；省份点击热区与详情图片；本地表标识关联；城市、驿站和军队图标；文本内容、横竖排、多行、旋转与整数/小数 zoom 连续缩放；同坐标城市点与非空文本在最小 zoom 与缩放结束后的中心重合；文本区域后的省份 hover/click/contextmenu 穿透；政权驻军图标与缺图回退、地块归属变化后的图标切换；标准视图配置色 / fallbackColor / noDataColor；无效 GeoJSON 是否触发官方整图回退。

Preview 不能确认：真实省份状态、服务端命令、军队移动、回合结算和 Workshop 场景加载——这些移交「打包版 + Workshop 测试场景」。

打包版检查清单（Windows 打包版，发布前逐项过）：启用 Mod 并重启 → `modasset://` 命中底图、GeoJSON 和图标 → 未覆盖资源从 ASAR 正常回退 → 层级顺序（底图/省份/命令层 → 文本 → 船只 → 城市 → 驿站 → 驻军，Tooltip/Popup/页面 UI 在上）→ 文本和船只不拦截鼠标 → Tooltip 纯文本（`<script>` 等按字面显示）→ 连续修改军队/政权/省份详情后动态图层刷新无闪烁 → 大小写、扩展名和 Windows 路径 → 禁用 Mod 并重启确认恢复官方地图。注意：本地 `local_` Mod 的业务请求仍使用官方场景，打包版只证明客户端覆盖链路；业务正确性必须 Workshop 服务端场景新建存档验收（不要用旧存档判断修改后的初始数据）。
