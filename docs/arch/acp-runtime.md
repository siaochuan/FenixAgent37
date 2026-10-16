---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# ACP / Runtime 不变量

### ACP / Runtime

- `acp-link` 本地 WebSocket 始终需要认证，relay token 由启动流程从 stdout 获取，不得硬编码或记录。
- 服务重启不会自动清理旧 `acp-link` 进程；排查 `EADDRINUSE` 时先确认残留进程和端口归属。
- relay 断连只关闭连接，不等于终止 Agent 子进程；实例释放必须走对应生命周期管理。
- relay 必须传递 Agent `status`，前端依赖 `status.capabilities` 决定可用能力。
- ID 体系不可混用：ACP session ID 为 `ses_*`；RCS session ID 由 `createDeterministicRcsSessionId(agentId, userId, instanceUid?)` 生成，格式为 `rcs_*`（第三段实参是 `instanceUid`，形参名 `sessionId` 为历史遗留叫法）。多实例隔离靠该确定性 key：不同实例 → 不同 `rcsSessionId` → 不同 YJS doc；前端路由段 `/agent/chat/{agentId}/{sessionId}` 承载的同样是 `instanceUid`，不存在可用的 DB 会话 id（详见 `docs/arch/19-yjs-chat-streaming.md` §3.2 / §4.1）。

