---
description: 对当前 Mod 项目跑全量校验（validate），按四件套口径呈现结果与下一步
argument-hint: [mod 目录]
---

对用户指定的 mod 目录（缺省时询问用户项目路径）调用 MCP 工具 `history-simulator-mod-toolkit` 的 `validate`（大剧本可传 limit=50 先看规模），然后：

1. 报告 `valid`、`errorCount`/`warningCount`/`issueCount` 与 `contractCoverage.missing`（缺表清单）；
2. error 逐条给 `code`/`reasonCode` 与位置，需要修法时查 `codes` 工具（按 reasonCode 取修复指引），不要 grep 源码；
3. 提醒：validate 0 error 只是四件套第一件——交付前还需 preflight valid、quality-gate 未裁定 0 且已裁定无 error、缺表 0；
4. 不要自行修改文件，先给修复计划等用户确认。
