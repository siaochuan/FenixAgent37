---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# YJS / Chat 不变量

### YJS / Chat

以下约束共同保证刷新恢复、多标签页一致性和消息不重复：

1. `rcsSessionId` 必须确定性生成，不得使用 `Date.now()` 或随机值，否则刷新后旧 Y.Doc 不可达。
2. WebSocket open 时必须在 `relayReady = true` 前发送 Chat Doc 和 Session Doc 初始快照。
3. 重连时必须从 `chatMeta.activeSessionId` 恢复 `entry.acpSessionId`；同一 ACP session 的 `load_session` 必须跳过 Agent 全量回放。
4. `session/list`、`session/new`、`session/load` 和 `session/resume` 的 `cwd` 必须由服务端 translator 注入；Agent status 到达前不得发送 `list_sessions`。
5. Y.Doc 名称使用 `chat:{rcsSessionId}` / `session:{rcsSessionId}`，广播必须按 `rcsSessionId` 隔离，禁止全局广播会话数据。
6. 用户消息只由后端写入 Y.Doc；前端不得维护第二份 `localUserEntries`，否则 Agent 回显会造成双写。
7. 会话切换与内容清理走"换代"：由 `DocManager.replaceProjection` 完成；不得通过 destroy + recreate 制造异步竞态。`chat-writer.ts` 的 `clearSessionDocContent`（及专用的 `clearPeriTaskViews`）已删除，不要恢复旧的清空流程（`docs/arch/19-yjs-chat-streaming.md` §4.2 / 前端规范 §8.5 第 7 条）；`create_session` 同样走换代。
8. 同一 `instanceId + userId` 的多标签页共享一个 relay handle；引用计数归零后才释放，切换 session 时同步同组客户端的 `acpSessionId`。
9. WebSocket 发送背压阈值为 64 KB，默认连接上限为 200（`YJS_MAX_CLIENTS`）；修改时必须保留限流、资源释放和单连接故障隔离。
10. `ChatView` 与 `EntryRenderer` 使用 `React.memo`；comparator 必须与调用方 prop 稳定性保持一致，修改 props 时同步更新 comparator 和相关渲染测试。
11. `@fenix/chat-channel` 根入口必须浏览器安全：只导出类型、schema、`chat-writer`、`yjs-store`、`protocol`、`transport`、`util`；服务端能力（`channel` 控制面、`persist` 持久化、`state` 聚合层 DocManager/factory/aggregator 等）必须经 `@fenix/chat-channel/server` 子路径导出。前端消费方经包 `exports` 直连根入口；从根入口 re-export 服务端模块会把 Node 依赖打进浏览器 bundle 并导致加载失败，边界由 `packages/chat-channel/src/__tests__/chat-channel-browser-surface.test.ts` 静态走值导入图守护。

