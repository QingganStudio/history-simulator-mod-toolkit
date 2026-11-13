---
description: 对当前 Mod 项目跑发布前内容预检（preflight，与游戏上传门禁逐码一致）
argument-hint: [mod 目录]
---

对用户指定的 mod 目录（缺省时询问用户项目路径）调用 MCP 工具 `history-simulator-mod-toolkit` 的 `preflight`，然后：

1. 报告 `valid` 与 issue 清单（severity + code/reasonCode + 位置）；
2. 上传零容忍：warning 级同样拦截，必须修到 issues 清空——逐条给修法（查 `codes` 工具按 reasonCode 取修复指引）；
3. preflight valid 后提醒用户：这是四件套第二件，交付前还需 validate 0 error、quality-gate 未裁定 0 且已裁定无 error、缺表 0；
4. 不要自行修改文件，先给修复计划等用户确认。
