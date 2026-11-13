---
title: 13.6 省份指令编辑器
description: 配置省份管理命令的展示、值类型、作用范围和 AI 文案
---

# 13.6 省份指令编辑器

省份指令编辑器维护省份管理菜单的客户端命令 Schema。它决定按钮/输入的显示语义、默认值、适用省份和提示文本；它不实现服务器端命令、不会让本地 Mod 获得真实场景，也不修改省份 CSV 数据。需要调整地理数据请编辑地图/省份资源；需要验证命令真正执行必须在 Workshop 场景完成。

## 上传预检结构校验

整目录预检对本文件执行字段结构校验（`json-schema` 码）：顶层必须是含 `version`（字符串）与 `commands`（非空数组）的对象；每条命令的 `command` / `command_name` 非空、`unit` 字符串、`valueType` 限 number 或 boolean 且 `defaultValue` 类型必须与之一致、`extra` 对象、`scope` 限 mainland / non-mainland / all——拼错字段名或类型不符会在上传前被拦下。

## 输出与字段

保存输出 `client-assets/map-data/province-schema.json`，不进入 Manifest。文件的 `commands` 数组中，每项包含以下字段：

| 字段 | 说明 |
| --- | --- |
| `command` | 稳定命令键 |
| `command_name` | 菜单显示名称 |
| `unit` | 数值单位/显示单位 |
| `valueType` | `number` 或 `boolean` |
| `defaultValue` | 默认输入值 |
| `scope` | 可选：作用范围 |
| `ai_description` | 提供给 AI/命令语义的描述 |
| `display_description` | 给玩家看的操作说明 |

例如：

```json
{
  "commands": [
    {
      "command": "relief_fund",
      "command_name": "赈灾拨款",
      "unit": "两",
      "valueType": "number",
      "defaultValue": 0,
      "scope": "mainland",
      "ai_description": "向{target_name}调拨{value}{unit}用于赈灾。",
      "display_description": "向目标省份调拨赈灾款项。"
    }
  ]
}
```

新建命令时编辑器默认 `scope: "mainland"`；运行时旧文件缺少 `scope` 时按 `all` 处理。对于省份属性，`is_main === false` 视为 `non-mainland`，其他值均视为 `mainland`。

## 操作步骤与模板变量

1. 在侧栏打开“省份指令编辑器”，新增或选择一条命令。
2. 选择 `number` 或 `boolean`，并让 `defaultValue`、单位和描述符合该输入方式。
3. 选择作用范围并明确写出 `scope`，不要依赖缺失时的 `all`。
4. 在 AI 描述中按需使用快捷模板变量，保存后重载省份管理页检查显示。
5. 在适用的省份类型中分别检查菜单是否出现、默认值是否正确。

编辑器支持的快捷变量为 `{target_name}`、`{command_name}`、`{value}`、`{unit}`、`{target_id}`。未知变量在运行时替换为空字符串，所以拼写错误往往不会造成文件加载失败，却会留下缺词的命令文本。

## 运行时校验、刷新与边界

运行时只要求 `commands` 非空，并要求每项具有 `command`、`command_name`、`valueType`；不满足时整份文件回退官方默认 Schema。它不会替你保证文案准确、命令键唯一、默认值能被后端执行或 `scope` 设计合理。

该文件不在 Mod Maker 的 `PREVIEW_JSON` 白名单中。它不会因为未保存快照而在 Preview 中热更新；请先保存，再重载相关游戏页面。这样可以验证菜单和客户端元数据，却不等同于验证服务端下令成功。本地 `local_` Mod 仍使用官方服务端场景。

## 提交前检查

- `commands` 不为空；每项有稳定命令键、显示名和值类型。
- `valueType`、默认值、单位和文案彼此一致。
- 作用范围已显式选择，并分别检查大陆与非大陆省份。
- 只使用五个已支持的模板变量，且预览实际替换后的句子。
- 保存后重载目标页；服务端命令结果留待 Workshop 场景验证。

相关主题：[地图与省份](/resources-and-ui/map-and-provinces)、[历法配置](./calendar-config)、[本地测试边界](/validate-and-publish/local-testing)。
