---
description: 安装自检——验证 history-simulator-mod-toolkit MCP server 可用、工具面完整、捆绑数据自检双绿
---

不接任何用户 mod 目录，按以下顺序自检并逐项报告 通过/失败：

1. 调用 `history-simulator-mod-toolkit` 的 `schema_list`：应返回 {{TABLE_COUNT}} 张表与注册 JSON 逻辑名清单；
2. 调用 `codes`（无参）：应返回 {{ERROR_CODE_COUNT}} 个错误码目录；
3. 调用 `docs_search`（关键词「税」）：应 `totalCount > 0` 且返回命中；
4. 调用 `vanilla_table`（table=random_events，不传 row）：应返回完整行键索引（rowKeys 数量 > 100）；
5. 结论给用户：MCP server 是否健康、工具面是否完整；任何一步失败时给出排查建议（检查插件安装 / 重启会话 / 查看服务器日志）。
