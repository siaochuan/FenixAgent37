---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# Workspace 与远程文件

### Workspace 与远程文件

- workspace 路径运行时通过 `resolveWorkspacePath(organizationId, userId, environmentId)` 计算：`{WORKSPACE_ROOT}/{organizationId}/{userId}/{environmentId}`。
- DB `workspacePath` 是历史字段，不得用于推导真实目录。
- 浏览器传入的 workspace/cwd 不可信；ACP action 的 `cwd` 必须由服务端根据已认证 environment 注入。
- 文件 API 使用 RCS session/environment 上下文，不得把 ACP `ses_*` 当作 RCS 标识。
- `getRemoteMachineId` 优先读取 agent config 的 `machineId`，否则 fallback 到 `RCS_DEFAULT_MACHINE_ID`；无 `agentConfigId` 的 ACP/Bridge environment 同样必须执行 fallback。
- 配置了远程 machine 但 file-ws 未连接时必须返回明确错误，不得静默回退到本地 FS，避免远程/本地文件分裂。
- 文件 API 路径为 workspace 根相对路径，必须经过词法校验（绝对路径 / `..` / 控制字符）与 realpath 越界检查（symlink 逃逸防护）。

