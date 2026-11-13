---
title: 7.3 大臣、官职与举荐
description: 配置普通大臣、举荐候选、官职显示和自动派生输出
---

# 7.3 大臣、官职与举荐

人物系统由普通大臣、举荐候选、举荐触发数据、官职配置和四类人物图片共同组成。只完成其中一张表，通常只能看到不完整的人物卡片。

## 数据链路

```text
parties
  ├─ ministers
  └─ recommend_ministers ← recommend_base
         │
         └─ 保存生成 global/recommend.csv

ministers + recommend_ministers
  └─ 保存生成 global/minister_assets.csv
```

## 普通大臣

在 [`ministers`](/reference/tables/ministers) 中先填写稳定 `code_name`、姓名、党派和当前 Schema 要求的能力或状态字段。党派字段引用 `parties.code_name`。

官职显示和分组来自[13.2 官职编辑器](/advanced/game-and-ui-config/minister-positions)。官职配置不会自动创建大臣，大臣记录也不会自动创建新的官职定义；两边必须分别维护。

## 举荐候选与举荐基础

制作顺序如下：

1. 在 `recommend_ministers` 创建候选大臣并选择合法党派。
2. 在 `recommend_base` 新增举荐基础记录。
3. 让 `recommend_base.code_name` 与目标候选的 `code_name` 完全一致。
4. 填写举荐轮次、展示文字和选项文本。
5. 保存，让 Mod Maker 生成最终的 `global/recommend.csv`。

生成时，候选大臣的当前 Schema 字段会序列化到 `option_effect`。如果基础记录找不到同标识候选，该记录会被跳过；不要直接编辑派生文件补救。

## 人物图片和立绘标识

四类图片统一在左侧「立绘表」编辑器中管理：两表派生出的立绘行为只读（自动并入、不可删），也可新增两表之外的额外立绘行；额外行名称撞派生名时会被自动并入只读行。

普通大臣和举荐大臣共用 `dynamic-assets/memorial-cards/` 命名空间，每人可以有：

- `-chat`：聊天图；
- `-face`：头像；
- `-card`：立绘卡；
- `-argue`：廷议图。

资源标识由 `code_name` 去掉开头的 `数字_` 得到。例如 `1001_zhangsan` 使用 `zhangsan-card.webp`。两个大臣去前缀后不能得到相同资源标识，否则文件会发生冲突。

完整资源规则见[11.2 注册图片的目录与命名规则](/resources-and-ui/images-and-assets/registered-images)。

## Preview 行为

大臣页面会组合大臣、党派和省份；举荐 Mock 会组合 `recommend_base` 与 `recommend_ministers`，并从有效候选中选择记录。Preview 能检查人物名称、党派、卡片资源和举荐展示，但不能证明服务端真实轮次、概率或人物入仕结果。

## 修改和删除

- 修改显示姓名：通常不影响 `code_name` 引用，但要检查文案和图片语义。
- 修改 `code_name`：同步修改举荐基础、图片文件名及所有事件或效果引用。
- 删除候选：先删除对应 `recommend_base`，否则派生时会跳过。
- 删除普通大臣：检查职位、事件、推荐文案和人物资源是否仍被使用。
- 删除大臣：对应派生立绘行随之消失；曾因同名被并入的额外行不会恢复。

## 验证清单

- 普通和举荐大臣引用的党派都存在。
- 举荐基础与候选能按 `code_name` 一一匹配。
- 保存后两个派生 CSV 已更新且没有跳过预期记录。
- 四类图片能按去前缀后的标识加载。
- Workshop 场景中验证真实举荐轮次和入仕效果。

下一篇：[7.4 省份、土地、建筑与财政](./provinces-and-economy)。
