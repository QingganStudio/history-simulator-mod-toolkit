---
title: 13.3 全局配置
description: 正确维护诏书开场语和主政权标识，并区分编辑器、Preview 与运行时解析
---

# 13.3 全局配置

全局配置是一个小而严格的运行时 JSON：它只定义诏书开场语和主政权代码。它不用于创建政权数据、不替代 `regimes` CSV，也不控制服务端初始化。修改政权名称、属性或数据行时，应编辑 `regimes`；本页只处理页面显示和 Preview 场景选择所需的两个键。

## 上传预检结构校验

整目录预检对根目录 `global_config.json` 与 `client-assets/global_config.json` 两个入口都执行字段结构校验（`json-schema` 码）：顶层必须是对象，`edict_opening_phrase` 与 `main_regime_code` 必填且为非空字符串——缺失、类型不符或空串会在上传前被拦下。此外预检还会执行跨表匹配校验（`json-schema` 码、`json-fk-target-missing` 项）：`main_regime_code` 必须能在 `regimes.code_name` 中找到对应政权，失配会在上传前被拦下。未知字段按运行时口径放行（运行时丢弃，不报错）。

## 输出与正式契约

Mod Maker 编辑和导入只读取 Mod 根目录的 `global_config.json`。保存时会用同一次序列化结果生成字节一致的 `client-assets/global_config.json`，根目录文件仍以 `files.global_config` 登记，客户端副本不进入 Manifest。即使两份文件被外部改成不同内容，重新导入时也不会用客户端副本覆盖或回退根目录权威文件。正式运行时只接受以下两个字段：

| 字段 | 必须满足 | 消费者/含义 |
| --- | --- | --- |
| `edict_opening_phrase` | 非空字符串 | 客户端 `ImperialEdict` 的诏书页头 |
| `main_regime_code` | 非空字符串，且精确匹配一条 `regimes.code_name` | 主政权标识；Preview 用它选择场景 Mock |

最小正确文件：

```json
{
  "edict_opening_phrase": "奉天承运皇帝诏曰",
  "main_regime_code": "2001_ming"
}
```

运行时解析器采用“全有或全无”：任一字段缺失、不是字符串或 `trim()` 后为空时，整份配置回退默认值；未知字段会被丢弃。不要把空字符串当作“隐藏标题”的开关，也不要依赖额外键携带业务数据。

## 编辑与 Preview 步骤

1. 在侧栏打开“全局配置”：诏书开头语为必填文本框；主政权从政权表（`regimes`）生成的下拉列表中选择，列表项同时展示政权名与 `code_name`。
2. 若当前值指向已被删除或改名的政权，编辑器会在字段上标记“未匹配”并阻止保存；问题面板同步给出可跳转的错误条目。
3. 保存 Mod；在 Preview 中查看诏书标题和主政权相关 Mock 是否选择了预期数据。
4. 在本地打包版验证 `client-assets/global_config.json` 的客户端资源加载；根目录 Manifest 文件的服务端消费仍应在 Workshop 场景验证。

编辑器在保存时执行与上传预检同口径的校验：两个正式字段必填非空（与运行时“全有或全无”解析一致），且 `main_regime_code` 必须匹配 `regimes.code_name`（`json-schema` / `json-fk-target-missing`）。经编辑器保存的 Mod 不会再出现失配；只有在编辑器外部手改文件时才需自行核对。

即便正式键已写对，Preview 的主政权代码若没有匹配 `regimes.code_name`，它会落到第一条政权记录。看到“有页面数据”并不证明选中了你想要的政权，必须检查实际 `code_name` 与 Preview 结果。

## 验证边界与检查清单

- Mod Preview 可验证诏书页头和有限场景 Mock；它不连接真实服务端。
- 本地 `local_` Mod 使用官方服务端场景，不能证明主政权已被服务端接纳。
- Workshop 场景才能验收真实场景初始化、跨回合状态与服务端规则。

提交前请确认：只有两个正式字段；两者都是非空字符串；`main_regime_code` 精确匹配某条 `regimes.code_name`（经编辑器下拉选择即天然满足；绕过编辑器手改文件时需人工核对）；并已用适当环境验证回退是否符合预期。

相关主题：[官职编辑器](./minister-positions)、[政权表参考](/reference/tables/regimes)、[保存与内容预检](/validate-and-publish/save-and-preflight)。
