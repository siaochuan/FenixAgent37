# 文件规模例外台账（AGENTS.md R-A3，只减不增）

`check:file-size`（待建）读取本表：未登记且超过 500 行的非测试、非生成源文件即失败；已登记文件行数不得超过登记值；新增条目需裁决。

| 文件 | 登记行数 | owner | removeWhen | 理由 |
| --- | --- | --- | --- | --- |
| packages/acp-link/src/server.ts | 1845 | <待指定> | 拆分为连接管理/鉴权/转发模块后 | 存量 |
| packages/resources/knowledge/src/server/services/knowledge-provider/ragflow.ts | 1469 | <待指定> | 按 RAGFlow API 分组拆分后 | 存量 |
| …（2026-10-15 克隆实测共约 40 个超限文件，首次落地时由脚本生成完整清单） | | | | |
