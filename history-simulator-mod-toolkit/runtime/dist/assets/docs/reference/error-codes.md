---
title: 第 23 章：错误码总目录
---

> 适用范围：mod-toolkit 错误目录全部 47 个顶层错误码（`validate` / `preflight` 报告的 `code` 全集）；最近代码核对：2026-10-09；页面类型：错误码权威清单。
> 数据源：共享错误目录（三方逐字锁定）；第一、二节目录表由 `scripts/generate-error-codes-doc.mjs` 从契约生成（`GENERATED:error-catalog` 标记块内勿手改），契约增删码而生成脚本未同步即构建红。

## 为什么 severity 不够：先懂三态

`severity`（error / warning）只表达「错误还是提示」，表达不了两件更要紧的事：**这个码会不会拦上传**、**服务端是否另有等价检查**。同一批 issue 按下面三态归位后，修复优先级才有依据。

> 方法论教训（两次实战总结）：官方内置剧本不走上传门禁——「官方镜像也报这个码」证明不了「这是误报」；判断危害看分区，不要看官方是否也报。

- **上传阻断**：`preflight`（= 上传门禁内核）会产出的码。上传**零容忍**——warning 级同样拦截，要修到 issues 清空而不是只清 error。
- **纯本地卫生**：只有 `validate` 会产出的码，不拦上传，服务端也无已知等价检查；但「不拦截」不等于「不重要」（见第三节）。
- **盲区警示**：客户端「全绿」不等于服务端必收——有些领域服务端另行强制，甚至有客户端暂无对应码的检查（见第三节后的盲区表）。

<!-- GENERATED:error-catalog:start -->

## 一、上传阻断（40 码）

按领域分组；「severity」列以码级声明为准——目录对 7 个 warning 码显式声明，其余为 error 缺省（目录未显式写出即 error）；契约里带 reason 级 warning 的码在同一列追加注明。查某个码的全部 reason 与修复指引：`codes --code <码>`。

### Mod 根与物理结构（6 码）

| 码 | severity | 含义 |
|---|---|---|
| `root-invalid` | error | Mod 根目录无效 |
| `path-link` | error | Mod 中包含链接 |
| `path-special-file` | error | 包含非普通文件 |
| `mod-oversize` | error | Mod 总大小超过 1000MB |
| `file-changed` | error | 预检期间文件发生变化 |
| `file-invalid-utf8` | error | 文件不是有效 UTF-8 |

### manifest 与注册文件（7 码）

| 码 | severity | 含义 |
|---|---|---|
| `manifest-missing` | error | 缺少 manifest.yaml |
| `manifest-oversize` | error | manifest.yaml 超过 50MB |
| `manifest-invalid` | error | Manifest 结构无效 |
| `registered-file-missing` | error | Manifest 注册文件不存在 |
| `registered-file-type` | error | 注册文件类型不匹配 |
| `registered-file-duplicate-physical` | error | 多个注册项指向同一物理文件 |
| `file-oversize` | error | 注册文件超过字节上限 |

### CSV 结构与数据面（7 码）

| 码 | severity | 含义 |
|---|---|---|
| `csv-syntax` | error | CSV 语法错误 |
| `csv-metadata` | error | CSV 前 4 条 metadata 无效 |
| `csv-column-count` | error | CSV 数据列数不一致 |
| `csv-row-limit` | error | CSV 数据行数超限 |
| `csv-cell-limit` | error | CSV 单格超过 10,000 字符 |
| `csv-schema` | error | CSV 字段不符合 Schema |
| `text-char-limit` | warning | Markdown/TXT 超过 20,000 字符（>18,000 时 validate 侧另有 approaching-limit 余量预警） |

### JSON / YAML（5 码）

| 码 | severity | 含义 |
|---|---|---|
| `json-syntax` | error | JSON 语法错误 |
| `yaml-syntax` | error | YAML 语法错误 |
| `json-entry-limit` | error | JSON 条目数超过 1,000 |
| `json-schema` | error | JSON 结构不符合契约 |
| `asset-json-invalid` | error | 客户端 JSON 资源语法错误 |

### prompt 层与变量（4 码）

| 码 | severity | 含义 |
|---|---|---|
| `prompt-variable-missing` | error | 合成 Prompt 缺少必填变量 |
| `prompt-variable-undeclared` | warning | 合成 Prompt 存在未声明变量 |
| `prompt-structure` | error；reason 级 warning ×1：inert-logical-name | prompt 分层声明违反结构契约（后缀/子目录/越组/YAML 禁分层；镜像服务端写前闸） |
| `scenario-meta-schema` | error | ScenarioMeta 违反服务端 schema（根节点须为键值映射；13 字段严格类型——seasons 恰 4 / start_year 严格整数；镜像审核阶段 AUDIT_REJECTED） |

### 资产映射与预算（8 码）

| 码 | severity | 含义 |
|---|---|---|
| `asset-config-invalid` | error | assets.json 配置无效 |
| `asset-path-invalid` | error | 资源映射路径不安全或目标无效 |
| `asset-extension-unsupported` | error | 资源扩展名不支持 |
| `asset-family-mismatch` | error | 资源类型不兼容 |
| `asset-duplicate-source` | error | 资源 source 重复映射 |
| `asset-file-oversize` | error | 客户端资源文件过大 |
| `asset-image-invalid` | error | 图片无法读取 |
| `asset-image-pixels` | error | 图片超过 64MP |

### 立绘与 custom.css（3 码）

| 码 | severity | 含义 |
|---|---|---|
| `portrait-name-invalid` | error | 立绘表额外行名称非法 |
| `portrait-name-duplicate` | error | 立绘表额外行名称重复 |
| `custom-css` | error | custom.css 超过 1MB 大小上限 |

## 二、纯本地卫生（7 码）

| 码 | severity | 含义 | 为什么重要 |
|---|---|---|---|
| `asset-reference` | error；reason 级 warning ×4：province-not-on-map、regime-color-missing、regime-color-stale、position-name-duplicate | 跨文件引用完整性（双副本分歧 / 文件未登记 / 新增实体资源缺失） | 未登记文件不会被游戏加载；缺资源运行时静默破图 |
| `derived-table` | error | 派生表与源表不一致 | 派生表漂移 = 大臣两表改了但 minister_assets 没重建 |
| `csv-import` | warning | 导入层诊断（缺表 / 放错目录 / 元数据块 / 表头 / 单元格） | 缺表时游戏按空表处理——曾对校验完全不可见的假绿源头 |
| `prompt-layer` | warning | setting 层缺失或空白 | 留空时 AI 按原版崇祯世界观推演，剧本内容与作者意图脱钩 |
| `field-definitions` | warning | 字段覆盖与官方基线不符 | 写了不在基线的 schema/字段 = 运行时静默忽略，等于没写 |
| `prompt-render-residual` | warning（廷议条目为 error） | 层文件存在渲染后无法解析的 {{ }} 变量（廷议严格渲染；static 条目 fail-open 原文照发） | 拼错变量名会把未渲染原文直接发给模型 |
| `scenario-meta-unknown-key` | warning | ScenarioMeta 顶层键不在已知集（13 schema 字段 ∪ 原版键集） | 增键合法（服务端 extra=allow），但确认有后端消费方，否则是死配置 |

以上 7 码只有 `validate` 会产出（validate-only，不进上传门禁），服务端也无已知等价检查——但**别当噪音**：`asset-reference` 与 `derived-table` 是 error 级，工作流质量标准四件套（validate 0 error 在内）会拦；这七族是本地唯一对整表缺失、资源未登记、派生表漂移、setting 层空白、字段覆盖错配、渲染残留与未知配置键发信号的检查。

<!-- GENERATED:error-catalog:end -->

## 三、盲区：客户端全绿 ≠ 服务端必收

「上传阻断」以客户端 preflight 为准；以下领域**服务端另有等价强制**（客户端过了、服务端还会再查一遍），以及一个客户端暂无对应码的服务端检查——修问题时不要拿「客户端没报」当挡箭牌：

| 领域 | 客户端 | 服务端 |
|---|---|---|
| manifest 严格契约 / 注册文件 / assets.json 映射 / 资源预算 / 注册 JSON 结构 | `preflight` 拦（第一节对应码） | ✔ 另有等价检查 |
| 必需逻辑文件（`requiredLogicalFiles`） | **暂无对应码**（三方契约 pending，未实现） | 契约未披露（启用与否属三方确认问题 2 的讨论范围） |
| CSV 数据面 | `validate` error + `preflight` 同码拦截 | 契约未披露（客户端看不到该层是否另有检查，不要反推「服务端不查」） |
| CSV 结构项（`csv-column-count` / 行数档 / 单元格上限） | 结构项由 `preflight` 拦截；`validate` 侧仅 `csv-syntax` error 与 `csv-import/ragged-record` warning 诊断 | 契约未披露 |

四道门（validate / preflight / 编辑器保存预检 / 服务端）的逐项对照见[发布门禁与 preflight](../tools-and-agents/preflight-and-publish)的检查集矩阵。

## 四、编辑器导入提示 ↔ CLI 码对照

同一件事常有三套词：编辑器打开 Mod 时说「缺少 X，已用内置版本回填」、`validate` 报 `csv-import` 族、`list-tables` 标「导入告警」。下表对齐三方用语与危害等级（等级口径同前三节）：

| 编辑器打开 Mod 时的提示 | 工具侧对应 | 危害等级 |
|---|---|---|
| `[导入] 缺少 <表>.csv，已按空表处理` | `validate`：`csv-import / missing-table`（warning）+ `contractCoverage.missing`；`list-tables` 标「缺失」 | 纯本地卫生——但缺表 = 游戏按空表跑，务必补齐 |
| `[导入] 缺少 <注册 JSON>，已使用默认值；保存时将写入默认内容` | 保存后仍缺失则 `preflight`：`registered-file-missing`（error） | 上传阻断——回填的默认内容是否合意要人工确认 |
| `[导入] <JSON> 解析失败，已保留原文等待修复 / 已替换为默认值` | `preflight`：`json-syntax`（error）；client-assets 侧 `asset-json-invalid`（error） | 上传阻断 |
| `[导入] 缺少 <prompt / text 层文件>，已用内置版本回填` | `validate`：`prompt-layer / setting-missing` 或 `setting-empty`（warning） | 纯本地卫生——回填的是原版内容，剧本世界观会被静默换成崇祯 |
| `[导入] 立绘表有 N 个重复的额外行名称，已自动去重` | `preflight`：`portrait-name-duplicate`（error） | 上传阻断——编辑器会自动修，CLI 侧不会替你修 |
| `[导入] 内置基线缺少 <文件>，已用磁盘/空内容兜底` | 视落盘结果落入上面对应码 | 编辑器环境异常信号，检查安装完整性 |
| `[导入] 检测到旧格式 prompt 文件` | 无 CLI 对应码（编辑器专属提示） | 仅编辑器——旧文件不会被加载，内容需迁移到新格式 |
| 编辑器保存预检的 issue | 同名码族（共享错误目录；校验器实现有意分叉，覆盖面与严格度可能不同） | 见第一节分区——编辑器放行 ≠ toolkit 放行 |

> 编辑器与 mod-toolkit 内核各有一套校验器是**有意分叉**（内核更严、带 severity 分级）；两边共享同一份错误码目录，但「编辑器说行」不等于「工具说行」——发布前以 toolkit `preflight` 判定为准。
