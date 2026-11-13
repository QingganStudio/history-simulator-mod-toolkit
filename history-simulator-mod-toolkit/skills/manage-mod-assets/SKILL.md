---
name: manage-mod-assets
description: "替换《历史模拟器：崇祯》Mod 的图片、音视频或字体资源。用户要换立绘、事件图、兵种图、界面素材或音效，或放进资源后游戏里没生效、需要排查覆盖是否被静默回退时使用；默认按官方请求键把文件放进 client-assets 同路径目录直接覆盖，不登记 manifest、不写显式映射。CSV/JSON 数据与注册配置的改动不走本技能；用户要的是生成/绘制图像本身（而非把已有素材放进 mod 或排查覆盖）时不适用。 English triggers: replace images/portraits/audio assets, swap assets, audio, music, fonts, event images, interface assets, asset override not working."
license: MIT
compatibility: Requires the mod-toolkit MCP server (server name "history-simulator-mod-toolkit") or the mod-toolkit CLI, installed and registered.
metadata:
  version: "1.7.0"
  author: 青干工作室
---

# Skill：按官方目录结构直接覆盖资源

触发：用户要求替换游戏图片、音视频、字体等资源（立绘、事件图、兵种图、界面素材、音效）。

本技能需要宿主文件编辑能力；将用户指定的 Mod 根目录转为绝对路径再操作。

## 步骤

1. **确定官方请求键**：资源键 = 游戏请求的相对路径（目录、文件名、扩展名都要一致）。常见约定键：大臣立绘四件套 `dynamic-assets/memorial-cards/<去前缀名>-{chat,face,card,argue}.webp`；兵种图 `dynamic-assets/troops/<code_name>.webp`（势力变体 `<政权>_<兵种>.webp` 可选）；事件图分目录：常规事件 `dynamic-assets/events/normal/<code_name>.webp`；随机/固定回合/链式事件 `dynamic-assets/events/random/<code_name 点号换连字符>.webp`（`dragon.attack` → `dragon-attack.webp`）；省份浮层 `dynamic-assets/province-overlays/<code_name>.webp`；政权驻军图标 `dynamic-assets/garrison-icons/<code_name>.webp`。拿不准调用 MCP `docs_search` / `docs_page` 查资源文档，或读 `validate` 的 asset-missing 报错给出的键。
2. **同路径直接覆盖**：把合规文件放到 `client-assets/<请求键>`——相同相对路径、文件名和扩展名。普通 client-assets 文件不登记 manifest；`Config/assets.json` 保持空，目录约定命中即覆盖。
3. **确认索引真实命中**：调用 `asset_map`（传 `directory`）核对请求键出现在 `overrides` 且 `via` 为 `convention`、`rejected` 为空——索引未命中会静默回退官方资源，"页面能显示"不等于覆盖成功。
4. **格式不符先转换**：素材格式与请求键不同时优先转换素材（如 PNG 转 WebP 后同键放置）；只有同路径确实无法满足且作者确需该效果时，才在 `Config/assets.json` 加显式映射（`sourceFilePath` = 请求键、`targetFilePath` = `client-assets/...` 同资源族文件），并用 `asset_map` 确认 `via` 变为 `explicit`。转换是常规路径，映射是例外。
5. **复验**：调用 `preflight`（扩展名白名单、大小与像素预算、JSON 合法性必须全过）；被拒文件进 `rejected` 清单并回退官方资源，修正后重跑。

## 高危踩坑

- **静默回退不会报错**：键名打错、扩展名与请求键不符（如该 `.webp` 放了 `.png`）、文件超预算时，游戏静默显示官方资源——覆盖是否生效只看 `asset_map` 的 `overrides`，不看页面。
- **预算内才算合规**：图片 ≤32MB 且 ≤64MP、音频 ≤64MB、视频 ≤256MB、JSON ≤4MB；超预算进 rejected 并回退——先压缩/转码再放。
- **显式映射不是常规路径**：同族图片可换格式映射，JSON 只能映射 JSON；绝对路径、`..` 穿越、目录、跨族、重复 source 一律被拒。普通覆盖不写 assets.json。

## 检查点

- [ ] 替换文件与官方请求键同路径、同名、同扩展名
- [ ] `asset_map` 的 `overrides` 命中目标键（via=convention）且 `rejected` 为空
- [ ] `Config/assets.json` 保持空或不变（未动用显式映射例外）；manifest 未登记普通资源
- [ ] preflight valid
- [ ] 交付四件套分别报告：validate 0 error、缺表 0（contractCoverage.missing 为空）、preflight valid、质量门继承与新增 issue 分开报告、未代为裁定且已裁定无 error（acknowledged[] 无 error；需要作者选择的设计问题保持待处理）；汇报按「结论 → issue 定位 → 修复依据」组织，精简可追溯；四件套全绿仅证明静态交付——真实触发、结算与服务端场景验收移交「打包版 + Workshop 测试场景」人工环节

## English quick reference

**When to use:** replacing the mod's images, audio/video, or fonts (portraits, event art, troop images, UI assets, sound effects). Default path: place files under client-assets at the official request key — no manifest registration, no explicit mapping. Data/config changes belong to other skills.

**Workflow (MCP tool names; CLI uses hyphenated aliases):**
1. Determine the official request key (directory + filename + extension must match). Conventions: minister portraits `dynamic-assets/memorial-cards/<id-without-prefix>-{chat,face,card,argue}.webp`; troop images `dynamic-assets/troops/<code_name>.webp`; event art: normal events `dynamic-assets/events/normal/<code_name>.webp`, random/fixed-turn/chain events `dynamic-assets/events/random/<code_name with dots → hyphens>.webp`; province overlays; garrison icons. When unsure, `docs_search` / `docs_page`, or read the asset-missing error from `validate`.
2. Drop-in override at `client-assets/<request-key>` (same relative path/name/extension). Ordinary client-assets files are NOT registered in the manifest; keep `Config/assets.json` empty.
3. Verify real index hits: `asset_map` must list the key in `overrides` with `via: "convention"` and empty `rejected` — misses silently fall back to official assets ("it renders" proves nothing).
4. Format mismatch → convert the asset first (e.g. PNG→WebP). Explicit mapping in `Config/assets.json` is the exception path only.
5. Re-verify with `preflight` (extension whitelist, size/pixel budgets, JSON validity).

**Top pitfalls:** silent fallback never errors — trust `asset_map.overrides`, not the page; budgets: images ≤32MB/≤64MP, audio ≤64MB, video ≤256MB, JSON ≤4MB.
