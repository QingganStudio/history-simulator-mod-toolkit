---
name: custom-map
description: "创作《历史模拟器：崇祯》Mod 的自定义地图——GeoJSON 省份轮廓、城市、驿站、驻军与文本要素制作，底图与地名（text）策略决策，省份 code_name 全落点对账。用户要求改地图、把省份版图改成架空格局、换底图、增删省份轮廓与地名，或问地图没生效、整图回退官方时使用；省份业务数据行（数值、政权归属）增删走 extend-tables（扩表），省份浮层图片与底图文件等资源覆盖走 manage-mod-assets（资源），province-view-config 等注册 JSON 配置走 edit-mod-config（配置），修既有地图的校验报错走 repair-errors（修错）。 English triggers: custom map, change the map, alternate province borders, replace base map, GeoJSON provinces, map labels, map not applied."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：制作自定义地图

触发：用户要求"改地图 / 把省份版图改成架空格局 / 换底图 / 地图没生效"——GeoJSON 地图（省份轮廓、城市、驿站、驻军、文本要素）制作、底图与地名策略、省份 code_name 全落点对账。

## 步骤

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。GeoJSON 契约细表（根结构/环规则/属性结构）、五要素与业务绑定表、textStyle 白名单全表、保存规范化规则、code_name 落点对账清单、图标键与回退链、Preview 能力矩阵收在本技能 `references/map-contracts.md`——正文只留决策点，动笔前先读它。拿不准语义时调用 MCP `docs_page` 读 `resources-and-ui/map-and-provinces/` 下六篇。

1. **底图-text 决策点（工作流第一步）**：地名文字是否已印在底图上，决定要不要建 text 要素——沿用官方底图（1.6.0 起官方 `ming-map.webp` 已烘焙旧地名，原版 GeoJSON 已删除对应的 119 个旧 text 要素，照此为范本）→ 不建旧地名 text，否则 validate 报 legacy-map-text-overlap（叠字）；自制无字底图 → 自建全套 text（地名、区域名都要）；自制已印字底图 → 不建 text；底图没有的新地名永远合法（提醒只对身份、文字、坐标均匹配那 119 个旧要素的条目触发）。提供有效 `client-assets/home-map-assets/base/ming-map.webp` 自定义底图覆盖后，旧地名提醒自动关闭。
2. **数据先行**：省份业务记录（provinces 行、政权归属、土地/建筑/驻军）先按省份教程完成——地图编辑器不创建业务记录，code_name 规划先于动笔。调用 `schema_table` 查 provinces 字段、用 `vanilla_table` 对照原版省份记录；调用 MCP `vanilla_json`（key: map_province）取官方地图的结构摘要作参照（巨型 GeoJSON 返回摘要而非全文）。
3. **几何与坐标系**：CRS.Simple 平面坐标系，设计空间固定 1920×1080；GeoJSON 坐标按 [x, y] 存储（Leaflet 显示时才转 [lat, lng]）——真实经纬度不可直接导入，也不要把 x/y 写反。`properties.type` 五要素闭集：province / city / station / troop / text（旧 army Point 不迁移）。Polygon 每个环 ≥4 点且首尾坐标相同（闭合）；多边形顶点允许轻微出血（原版沿海省有约 -10 越界），但点要素坐标必须在 0–1920 / 0–1080 界内；单地块最多 200 个存储点（含闭合点）；MultiPolygon 编辑器不导入——多块领地拆成多个 Polygon（可用相同 code_name，交互需单独验证）。
4. **text 要素契约**：必须是 Point + `type: text`；`name` 是显示内容，不是业务外键。上限：≤256 码点、≤4 行、单图 ≤300 个文本要素；textStyle 只接受白名单字段与值域（全表见 references）。
5. **code_name 全落点对账**：省份 code_name 的落点链 = provinces 表定义行 → 土地/建筑/驻军 FK（province_land_holdings、buildings、troops.location_province_id）→ 事件/决议/国策内联引用 → GeoJSON province 要素 → `dynamic-assets/province-overlays/<code_name>.webp` 详情图键。改名前必跑 MCP `rename`（dryRun: true，只读）拿按文件分组的全量落点清单（GeoJSON 省份要素在落点覆盖内），改完用 `references` 确认旧名零残留。
6. **双向对账码闭环**：每写一批调用 `validate`——json-fk-target-missing（要素 code_name 在 provinces 表无对应行，error，"图画得出、详情为空"）必须清零；province-not-on-map（表有行、图无要素）是 validate-only warning，有意留空的省份可忽略。legacy-map-text-overlap 同为 validate-only warning（不在 47 个预检契约码内、不拦上传）——但叠字是视觉事故，必须清零。error 清零后再跑 `quality_gate` 与 `preflight`。
7. **整图静默回退防御**：运行时以整张 `ming_provinces.json` 为单位回退官方地图（请求失败、JSON 解析失败、共享契约不通过——单个非法要素同样触发整图回退）。游戏里"页面正常显示"不算数：调用 `asset_map` 核对 `map-data/ming_provinces.json` 真实命中 overrides，未命中即仍在用官方图。
8. **图标与配色衔接**：城市/驿站全局图标 `home-map-assets/icons/city-icon.*` / `station-icon.*`；驻军图标运行时按省份当前政权请求 `dynamic-assets/garrison-icons/<regime code_name>.webp`（仅 WebP），缺失回退全局 `home-map-assets/icons/troop.webp`——地块易主自动切换，不用改 GeoJSON。势力配色在 `map-data/province-view-config.json`（标准视图键 = regimes.code_name）。放图与命中核对全流程走 manage-mod-assets 技能；province-view-config 等注册 JSON 的结构规则走 edit-mod-config 技能。
9. **验证边界交接**：Preview 可查底图/边界/点位对齐、省份点击热区与详情图、图标与回退、文本缩放、无效 GeoJSON 是否触发整图回退；不能确认真实省份状态、服务端指令执行与回合结算——移交「打包版 + Workshop 测试场景」人工验收（打包版检查清单见 references）。

## 高危踩坑（实战最高频坑，动工前通读）

- **整图静默回退是最高危**：地图契约任一要素不合法，运行时以整张为单位回退官方地图——游戏照常打开、页面照常显示，你的省份版图根本没加载。"能进游戏"零证据价值，必须用 asset_map 核对地图文件真实命中 overrides。
- **沿用官方底图还留旧地名 = 叠字**：1.6.0 官方底图已烘焙旧地名，原版 GeoJSON 删掉对应 119 个旧 text 要素就是范本；从旧剧本整份拷来 ming_provinces.json 会把这批旧文字叠回底图（legacy-map-text-overlap）。先走底图-text 决策点再动笔。
- **坐标系陷阱**：这是 CRS.Simple 平面坐标不是经纬度地图——真实经纬度直接导入全部错位；GeoJSON 存 [x, y]，把 x/y 颠倒写成 [lat, lng] 顺序同样全错。底图更换后拉伸到 1920×1080，不会自动缩放既有几何。
- **改 code_name 漏一处落点就是连锁崩**：GeoJSON 省份要素、土地/建筑/驻军 FK、事件决议国策内联引用、省份浮层图键全部引用同一 code_name——改名前必先 rename dry-run 拿全量清单，漏掉 GeoJSON 要素即"上图但详情为空"（json-fk-target-missing）。
- **点要素越界 / MultiPolygon 整份拒收**：导入与保存预检以整张文件为单位判定——含越界 Point、未知类型或 MultiPolygon 的文件被整体拒绝或触发整图回退，不会只跳过坏要素。

## 检查点

- [ ] 底图-text 策略已按决策点确定（沿用官方底图 = 不建旧地名 text；自制无字底图 = 全套 text；已印字底图 = 不建）
- [ ] provinces 业务记录先就位、code_name 规划完成（地图编辑器不建业务记录）
- [ ] 几何合规：FeatureCollection 根结构、五要素闭集、Polygon 环闭合 ≥4 点、点要素在 1920×1080 界内、无 MultiPolygon、单地块 ≤200 存储点
- [ ] text 要素 name 为显示内容（非外键）；≤256 码点 / ≤4 行 / 单图 ≤300 个，textStyle 全部白名单值
- [ ] code_name 全落点已同步（改名经 dry-run 落点清单）；validate 无 json-fk-target-missing
- [ ] legacy-map-text-overlap 清零（validate-only warning，不拦上传但叠字必须修）
- [ ] asset_map 核对确认 `map-data/ming_provinces.json` 真实命中 overrides（未静默回退官方）
- [ ] Preview 未验证项（真实省份状态 / 服务端指令 / 回合结算）已移交「打包版 + Workshop 测试场景」
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** the user asks to build or change the mod's map — GeoJSON province outlines, cities, stations, troop points, text labels, base-map and place-name strategy, or "the map didn't take effect / fell back to the official map". Province business rows (stats, regime assignment) belong to extend-tables; image/asset overrides (province overlays, base map file) belong to manage-mod-assets; registered JSON config (province-view-config etc.) belongs to edit-mod-config; fixing validation errors on an existing map belongs to repair-errors.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Base-map vs text decision FIRST: the official 1.6.0 `ming-map.webp` already bakes legacy place names and the official GeoJSON deleted the corresponding 119 legacy text features — if you keep the official base map, do NOT create text features for those old names (validate raises legacy-map-text-overlap, overlapping glyphs). Self-made wordless base map → author the full text set. Self-made base map with printed names → no text. New names not on the base map are always legal. A valid `client-assets/home-map-assets/base/ming-map.webp` override silences the legacy-name reminder.
2. Data first: provinces rows, regime assignment, land/buildings/troops exist before map work (the map editor creates no business records); plan code_name up front. `schema_table` for provinces fields, `vanilla_table` for official rows, `vanilla_json` (key: map_province) for the official map's structural digest (giant GeoJSON returns a summary, not full text).
3. Geometry and CRS: CRS.Simple, fixed 1920×1080 design space; GeoJSON stores [x, y] (Leaflet converts to [lat, lng] for display) — real lat/lng cannot be imported directly, never swap x/y. Closed set of five `properties.type`: province / city / station / troop / text (legacy `army` points are not migrated). Polygon rings need ≥4 points and identical first/last coordinates; polygon vertices may bleed slightly out of bounds (official coastal provinces do), but Point coordinates must stay within 0–1920 / 0–1080; ≤200 stored vertices per polygon (including the closing point); MultiPolygon is not imported — split into multiple Polygon features (same code_name allowed, test interactions separately).
4. Text contract: Point + `type: text`; `name` is the displayed content, NOT a business foreign key. Limits: ≤256 code points, ≤4 lines, ≤300 text features per map; textStyle accepts only whitelisted fields/ranges (full table in `references/map-contracts.md`).
5. code_name reconciliation chain: provinces definition row → land/building/garrison FKs (province_land_holdings, buildings, troops.location_province_id) → inline references in events/decisions/perks → GeoJSON province feature → `dynamic-assets/province-overlays/<code_name>.webp`. Before any rename run `rename` with dryRun to enumerate every landing spot (GeoJSON province features included), then `references` to confirm zero residue.
6. Two-way reconciliation codes: `validate` after every batch — json-fk-target-missing (feature code_name absent from provinces, error: outline renders, detail empty) must be zero; province-not-on-map (row exists, no feature) is a validate-only warning — intentional omissions may be ignored. legacy-map-text-overlap is likewise validate-only (not among the 47 preflight contract codes, does not block upload) — but overlapping glyphs are a visual defect, clear it to zero. Then `quality_gate` + `preflight`.
7. Silent whole-map fallback defense: at runtime the entire `ming_provinces.json` falls back to the official map as a unit (request failure, JSON parse failure, or shared-contract failure — one illegal feature falls back the whole map). "The page looks fine" proves nothing: confirm with `asset_map` that `map-data/ming_provinces.json` actually hits overrides.
8. Icons and colors: global city/station icons `home-map-assets/icons/city-icon.*` / `station-icon.*`; garrison icon is requested per current regime `dynamic-assets/garrison-icons/<regime code_name>.webp` (WebP only), falling back to global `home-map-assets/icons/troop.webp` — ownership flips switch icons without touching GeoJSON. Faction colors live in `map-data/province-view-config.json` (standard view keys = regimes.code_name). Placement and hit verification: manage-mod-assets; registered JSON structure rules: edit-mod-config (full tables in references).
9. Verification boundary: Preview proves base map/boundary/point alignment, province click zones and detail images, icon and fallback chains, text scaling, and whether invalid GeoJSON triggers whole-map fallback; it cannot prove real province state, server-side command execution, or turn resolution — hand off to manual acceptance on the packaged build with a Workshop test scenario (packaged-build checklist in references).

**Top pitfalls:** silent whole-map fallback (an illegal feature anywhere drops the entire map — verify hits with asset_map, never trust "page displays"); keeping legacy text on the official 1.6.0 base map (overlapping glyphs — run the base-map/text decision first); CRS traps (real lat/lng import, or swapping the stored [x, y] order); renaming a province code_name without syncing every landing spot (run rename dry-run first).
