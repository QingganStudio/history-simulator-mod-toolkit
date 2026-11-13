---
title: 13.2 官职编辑器
description: 编辑官职分组、职位展示名称和排序，并理解 Preview 的匹配边界
---

# 13.2 官职编辑器

官职编辑器维护可展示的官职分组与职位列表。它改变的是静态职位定义和数组顺序，不会任命真实大臣、修改 `ministers` CSV 的人物数据，也不会让本地 Mod 获得服务端场景。要调整具体大臣的资料，请编辑 `ministers` 表；要验证真实任命逻辑，必须等 Workshop 服务端场景。

## 上传预检结构校验

整目录预检对本文件执行字段结构校验（`json-schema` 码）：顶层必须是分组数组；分组必须有字符串 `group_name`、`group_code` 和 `positions` 数组；职位必须有字符串 `key`、`name`、`rank`，且 `official` 必须保持 `null`——结构不符会在上传前被拦下，报错 message 会指到具体分组与职位。

非空与唯一性仍不在阻断范围（与编辑器口径一致）：空名称、重复 `key`、重名职位仍可保存并通过预检，保存前的自查要求不变。

## 输出与结构

保存输出 `global/minister_positions.json`，并作为 Manifest JSON 登记。顶层为分组数组，每个分组包含：

| 层级 | 字段 | 说明 |
| --- | --- | --- |
| 分组 | `group_name` | 分组显示名 |
| 分组 | `group_code` | 稳定分组代码 |
| 分组 | `positions` | 职位数组；数组顺序就是展示/排序结果 |
| 职位 | `key` | 稳定职位键 |
| 职位 | `name` | 职位显示名称 |
| 职位 | `rank` | 品级/排序字段 |
| 职位 | `official` | 必须保持 `null` |

最小结构如下：

```json
[
  {
    "group_name": "内阁",
    "group_code": "cabinet",
    "positions": [
      { "key": "shoufu", "name": "首辅", "rank": "正一品", "official": null }
    ]
  }
]
```

不要把某位大臣对象写进 `official`。任命状态属于运行时数据，而不是 Mod 中的静态官职定义。

## 操作步骤

1. 在侧栏打开“官职编辑器”，先创建或选择分组。
2. 填写分组名称、分组代码，再新增职位并填写 `key`、`name`、`rank`。
3. 通过拖拽调整职位顺序；保存后该顺序会序列化为 `positions` 数组顺序。
4. 保持每个职位的 `official` 为 `null`，不要用编辑器外的 JSON 手工塞入人物。
5. 保存 Mod；如需查看页面效果，在 Mod Preview 中检查大臣 Mock 的职位展示。

## Preview 如何匹配职位

Preview 的大臣适配器读取 `ministers.position`，按英文逗号、中文逗号或分号拆分文本，然后以**职位 `name`**匹配本文件中的职位名称；它不会以 `key` 匹配。例如，CSV 里写的是“首辅；大学士”时，Preview 查找的是 `name: "首辅"` 和 `name: "大学士"`。

所以，改 `key` 不会单独改变 Preview 中的职位匹配；改 `name` 后，必须同步检查 `ministers.position` 的文本。名称不一致时，Preview 只会缺少对应 Mock 匹配，这不代表真实服务端的任命关系已被验证。

## 校验与回退边界

统一保存内容预检对官职配置执行**结构**校验（顶层分组数组、字段类型、`official: null`，见上节），但不阻断非空与唯一性问题：空名称、重复 `key`、重名职位仍能保存与上传。这不是“允许作为正式作品发布”的证明，仍需作者在保存前自行检查。

本地 `local_` Mod 不接入服务端真实大臣/官职数据；Preview 只验证页面 Mock。正式游戏中人物分配、服务端场景初始化和跨回合变化必须在 Workshop 场景验证。若文件解析或消费者无法使用配置，应按对应运行环境回退，不要把 Preview 的局部显示当作完整有效性结论。

## 提交前检查

- 每个分组都有显示名、稳定代码和职位数组。
- 每个职位保留 `key`、`name`、`rank`、`official: null`。
- 拖拽后的数组顺序符合预期；改了 `name` 时已核对 `ministers.position` 文本。
- 已主动检查空值和重复项，而非只依赖保存预检通过结果。
- 已区分 Preview Mock、本地客户端与 Workshop 服务端的结论。

相关主题：[全局配置](./global-config)、[大臣数据表](/reference/tables/ministers)、[预览边界](/validate-and-publish/preview)。
