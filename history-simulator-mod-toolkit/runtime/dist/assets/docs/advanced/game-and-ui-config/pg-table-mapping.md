---
title: 13.1 PG 表名映射
description: 使用 PG 表名映射编辑器维护固定表的导出别名和中文显示名
---

# 13.1 PG 表名映射

PG 表名映射为固定的数据库表维护导出时使用的别名与中文显示名。它只改变**导出显示映射**；不会重命名 CSV 文件、内部 logical name、CSV 字段、外键或游戏运行时的数据表。若你的目标是改变数据内容，请回到相应 CSV 表；若只是让导出名称更易读，才使用本编辑器。

## 输出与适用范围

保存后文件写入 Mod 根目录的 `pg-table-mapping.json`，并作为 Manifest JSON 登记。当前可编辑的 PG 表固定为以下 14 张：

`regimes`、`parties`、`social_classes`、`land_tax_configs`、`equipments`、`ship_configs`、`buildings`、`ministers`、`arm_configs`、`provinces`、`province_land_holdings`、`troops`、`finances`、`events_prediction`。

这不是“为任意 CSV 创建数据库表名”的工具。列表以编辑器内置的已知 PG 表为准，不能借由写额外键来扩展；编辑器恢复默认时会保留未知扩展字段，但 validate 与上传预检会把未知表键拦为 error（见下文 rename 联动），它不应成为作品契约。

## 编辑步骤

1. 在 Mod Maker 侧栏打开“PG 表名映射”。每一行对应一个固定 PG 表。
2. 仅在需要覆盖默认值时填写 `table_alias` 和/或 `local_cn`；未填写的部分继续使用默认映射。
3. 先处理行内报错，确保别名格式正确且没有与其他表重复。
4. 保存 Mod，确认 `pg-table-mapping.json` 已写入根目录并通过内容预检。
5. 如需恢复某行默认值，使用编辑器恢复操作；输出会保持稀疏覆盖，而不是复制整套默认映射。

## 字段规则与最小示例

| 字段 | 规则 | 用途 |
| --- | --- | --- |
| `table_alias` | 非空覆盖必须匹配 `^[a-z][a-z0-9_]*$`；留空表示使用默认值 | 稳定的英文 snake_case 导出别名 |
| `local_cn` | 必须是字符串；留空表示使用默认显示名 | 中文显示名 |

例如，只覆盖政权表和大臣表时，可得到类似下面的稀疏 JSON：

```json
{
  "regimes": { "table_alias": "regimes_export", "local_cn": "政权" },
  "ministers": { "local_cn": "大臣" }
}
```

每个非空覆盖的 `table_alias` 必须在**全部已知 PG 表（包含默认值）**中全局唯一。`regimes_export` 可以使用；`Regimes`、`regimes-export`、`1_regimes` 都不符合格式。即使你没有在当前页面手工填写另一行，也不能使用那一行默认值已占用的别名；想恢复默认时应留空或使用恢复操作，而不是填写重复别名。

## 保存级阻断与回退

这是本章各高级配置中唯一具有专门保存级业务阻断的配置。别名格式错误、中文名不是字符串或有效别名重复时，相关行会同时标出问题，不能把它当作普通预警绕过后保存。恢复默认时，编辑器会保留未知扩展字段，但运行时契约只应依赖上述两个字段。

它不参与游戏页面 Mock，因此 Mod Preview 不能展示“别名已经生效”的游戏界面。正确的验证链是：编辑器行级校验 → 保存与内容预检 → 需要导出/数据库消费的实际流程。不要用本地客户端页面或 Workshop 场景来代替表名映射的契约检查。

## rename 联动

映射的**键是固定表名**：schema 的表集合（含本编辑器的 14 张 PG 表）由平台定义，Mod 不改表名，因此本文件不会因表名变化而需要维护。行级 `code_name` 改名同样不影响本文件——映射值只包含 `table_alias` / `local_cn`，不引用任何行的 `code_name`；mod-toolkit 的 `rename --dry-run` 落点清单也不包含本文件。

反过来，多写一个不存在的表键不会静默：未知表键会被 `json-schema` / `unknown-table-key` 拦下（error 级，validate 与 preflight 双侧）——映射一张 schema 里不存在的 PG 表是死配置，不会生效。

## 提交前检查

- 只为 14 张已知 PG 表填写映射，未尝试借键名扩展表。
- 每个非空别名均为小写 snake_case，且没有与默认值或其他覆盖重复。
- 明确这是导出显示映射，未同步修改 CSV 名、字段名或外键。
- 已保存并确认 Manifest JSON 预检通过。

相关主题：[数据表规则](/mod-maker/data-rules)、[文件结构](/reference/file-structure)、[全局配置](./global-config)。
