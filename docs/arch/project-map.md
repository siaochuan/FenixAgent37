---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# 项目与架构地图

FenixAgent 是基于 Elysia + Bun 的多租户 ACP Agent 平台，前端使用 React 19 + Vite，数据层使用 PostgreSQL + Drizzle ORM。

- 主要能力：组织与多租户、Agent 配置、ACP 实时通信、工作流、知识库、定时任务和 IM 通道。
- 根目录 `package.json` 是前后端统一依赖清单；`apps/web/` 是前端应用入口、Vite 配置、前端业务实现与构建产物。
- `packages/` 是 Bun workspace，按职责分为四类：`platform/*`（`platform-sdk` 稳定契约 + `identity` / `access-control` 两个有状态平台实现）、`agent-runtime`、`resources/*`、以及独立 SDK/插件包。跨包能力应通过包导出的稳定接口复用，不得依赖包内实现细节；类别之间的允许方向见 `docs/developer/architecture/engineering-standards.md` §2.3 的依赖矩阵，由 `bun run check:dependencies` 与 `bun run architecture:check` 强制。

### 后端地图

- `apps/server/src/`：唯一进程宿主，只保留入口、协议聚合、认证 adapter、OpenAPI、横切能力、应用基础设施初始化与装配；不得承载资源领域规则。
- `packages/platform/*`、`packages/agent-runtime`、`packages/resources/*`：各自 owner 持有资源 Facade、Domain Service、Repository、route contribution、schema、测试和 Web contribution。
- 外部资源动作遵循 `route → Resource Facade → Domain Service / Repository`；跨包只经公开 export，详细分层与目录规则见后端开发规范。
- `apps/server/src/db/schema.ts` 仅保留身份表转出、`data_migrate_record` 与经裁定暂留的旧授权表；业务表定义在各自 owner 包的 `db/schema.ts`。

### 前端地图

- `apps/web/src/routes/`：TanStack Router 文件路由；`routeTree.gen.ts` 为生成文件，严禁手改。
- `apps/web/src/pages/`：页面和业务容器。
- `apps/web/src/shell/`：本版本最终 Shell（布局、导航容器、Provider、鉴权后壳）；通用 UI 与业务组件归 `@fenix/ui-components`，宿主不再保留副本。
- `packages/web-runtime/web/api/`：前端请求基建（`request<T>()` / `unwrap` / `ApiError`，经 `@fenix/web-runtime/api/*` 出口引用）。**API 建模层不在这里**——域模块按资源归属放在各 owner 包的 `web/api/`；宿主 `apps/web/src/api/` 只剩 `branding.ts` 一个宿主专有域（`file-events.ts` 与 `fs.ts` 归 `@fenix/resource-machine` 的 `web/api/`、`instances.ts` 归 `@fenix/agent-runtime`、`peri-task-details.ts` 归 `@fenix/model-management`、`helpers.ts` 已删除）。
- `apps/web/src/i18n/`：国际化配置与语言资源。
- `apps/web/src/__tests__/`：前端关键流程测试。

