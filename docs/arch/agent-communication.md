---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# Agent 通信权威路径

### Agent 通信权威路径

Agent 通信分为三种明确场景，底层 relay 与 ACP 消息规则必须复用，不得再创建独立 JSON-RPC 协议栈。

| 场景 | 权威实现 | 生命周期 |
| ------ | ---------- | ---------- |
| HTTP / 程序化单轮调用 | `packages/agent-runtime/src/routes/api/openai-chat.ts` → `packages/agent-runtime/src/services/agent-chat-service.ts` | `openAgentSession` 解析并确保当前用户的持久 `api/primary` Instance runtime；每次请求创建独立 relay/ACP session/turn，dispose 只释放请求资源，不停止 runtime |
| Workflow | `packages/resources/workflow/src/server/services/workflow/agent-chat-transport.ts` | 解析并确保当前用户的持久 `workflow/primary` Instance runtime，通过 lease 保护并发 run；每个节点使用独立 relay/ACP session/turn |
| 前端交互式 Chat | `packages/chat-channel/src/channel/`（`@fenix/agent-runtime` 装配 `packages/agent-runtime/src/server/services/chat-channel-bootstrap.ts`） | 使用共享 relay、Y.Doc 状态和独立 session 生命周期；复用 `connectAgentRelay` 与 `@fenix/chat-channel` translator |

- relay JSON-RPC 必须兼容原始 `{ jsonrpc: "2.0", ... }` 和包裹 `{ type, payload: { jsonrpc: "2.0", ... } }` 两种格式，统一使用现有 `extractJsonRpc()` 模式。
- `session/update` 的事件类型位于 `params.update.sessionUpdate`，事件载荷位于同一 `update` 对象，文本内容通常在 `update.content`；禁止读取不存在的 `update.agent_message_chunk` 或把 `sessionUpdate` 当作文本。
- 实例策略不可混用：HTTP 自动选择 `api/primary`，Workflow 自动选择 `workflow/primary`；交互式 Chat 通常连接前端显式携带且重新校验归属的持久 Instance，未指定时才自动选择 `chat/default`。三者均通过 `AgentInstanceRuntimeCoordinator` 确保 runtime，但请求/session 不拥有共享 runtime 生命周期。
- 新入口不得新建独立 `acp-transport.ts`，也不得复制 session/new、session/load、session/prompt 的完整协议流程。

