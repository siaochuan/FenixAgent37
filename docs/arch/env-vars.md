---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# 环境变量

## 环境变量

环境变量的类型、默认值和必填性有两处真相来源：**宿主自有变量与多模块共享键**以 `apps/server/src/env.ts` 为准；**有唯一模块 owner 的部署变量**以其 owner 模块的 `fenix.module.ts` 里的 `envDefinitions` 为准（如 agent-runtime 的运行态旋钮与 `WORKSPACE_ROOT`、knowledge 的 RAGFlow/Gotenberg、sandbox 与 model-management 的整族配置）。同名键不得两处声明——`assertNoHostKeyOverride()` 会在启动期直接拒绝。新增变量必须同步 schema、部署配置和相关文档，并改在它的 owner 模块。`RCS_YJS_SNAPSHOT_*` 三项由 agent-runtime 模块的 `envDefinitions` 声明，经宿主装配投影与 `ChatChannelDependencies.snapshotPersist` 注入 chat-channel 持久层；包内不再直读 `process.env`，也不再自带默认值（Redis 模式下缺注入即失败）。关键变量：

- 必填：`DATABASE_URL`、`RCS_API_KEYS`。
- 系统 API：`RCS_SYSTEM_API_KEYS`。
- 存储与密钥引用：`SKILL_DIR`（默认 `./data/skills`）、`WORKSPACE_ROOT`（默认运行目录下的 `workspaces`）、`RCS_REDIS_URL`；provider 等配置通过 `{env:RCS_SECRET_<name>}` 引用环境密钥。
- 系统管理：`RCS_SYSTEM_ADMIN_PASSWORD_FILE`（默认 `./data/password.txt`）。
- Agent 路由：`RCS_DEFAULT_MACHINE_ID`、`RCS_DEFAULT_ENGINE_TYPE`、`RCS_DISABLE_LOCAL_EXECUTION`。
- 观测透传：`LANGFUSE_PUBLIC_KEY`、`LANGFUSE_SECRET_KEY`、`LANGFUSE_BASE_URL` 由主服务声明，经 `launchSpec.env` 统一透传到 machine 上 agent 进程（peri 的 langfuse-client 直读同名变量）；未设置则不注入，`extraEnv` 同名变量仍优先。动态 user 维度：`LANGFUSE_USER_ID` 由 `buildAgentLaunchSpecForCore` 的 platformEnv 按实例注入（与 `USER_META_USER_ID` 同源：environment 属主优先，fallback 到实例用户），peri 的 langfuse tracer 写入 `TraceBody.user_id`。
- 并发与生命周期：`RCS_AGENT_MAX_CONCURRENCY`、`RCS_USER_AGENT_MAX_CONCURRENCY`、`RCS_SCHEDULED_AGENT_MAX_CONCURRENCY`、`RCS_ACP_IDLE_TIMEOUT_SECONDS`、`RCS_ACP_IDLE_SWEEP_INTERVAL_SECONDS`、`RCS_ACP_ACTIVITY_TIMEOUT_SECONDS`。Environment 不再持有独立并发配额；禁止在 `AgentController` 按 Environment 内存实例数重新引入限流。
- YJS：`YJS_MAX_CLIENTS`（默认 200；由 agent-runtime 模块声明并经 `AgentRuntimeModuleConfig.yjsMaxClients` 注入 chat-channel 装配）；快照持久化：`RCS_YJS_SNAPSHOT_INTERVAL_MS`（节流窗口，默认 2000）、`RCS_YJS_SNAPSHOT_IDLE_MS`（静默期，默认 500）、`RCS_YJS_SNAPSHOT_TTL_SECONDS`（快照滑动 TTL，默认 7 天）——三项同由 agent-runtime 模块声明，经 `AgentRuntimeModuleConfig.yjsSnapshot*` 投影进 `ChatChannelDependencies.snapshotPersist`。

