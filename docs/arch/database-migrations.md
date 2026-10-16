---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# 数据库与迁移

## 数据库与迁移

- **一张表的定义只在一个 owner 手里**：业务表定义在所属模块的 `packages/**/db/schema.ts`（经该包 `exports["./db"]` 公开，如 `@fenix/resource-task/db`），身份表在 `packages/platform/identity/db/schema.ts`；宿主 `apps/server/src/db/schema.ts` 只保留身份表转出、`data_migrate_record` 与仍在清理中的旧授权表。改哪张表就到它的 owner 包改，不得在宿主或别的包复制一份。`drizzle.config.ts` 的 `schema` 必须声明**全部** owner 包路径与宿主 schema，否则 `db:generate` 会把漏声明的一族误判为已删除。
- 表定义换手不得改变 DDL：`bun run check:schema-ddl-drift` 比对「`drizzle.config.ts` 声明的 schema 集合 → 最新 snapshot」的差异，必须为零。
- 跨包外键（表 A 的列引用别包表 B 的主键）只允许在 `db/**` 的**组装期**导入 B 的表对象（Drizzle `.references()` 只接受列对象），例外口径见 `docs/developer/architecture/engineering-standards.md` §6.1；`src/**`、`web/**` 的调用期跨包读表一律违规，必须改经该 owner 的公开服务端入口或宿主注入端口。
- 标准流程：修改 schema → `bun run db:generate --name <module>-<change>` → 审查 `drizzle/*.sql` 与 `drizzle/meta/*` → `bun run db:migrate` → 存量数据变更时执行 `bun run run-data-migrations` → 运行相关测试和 `bun run precheck`。
- 跨组织可见性由五张受控资源主表（`agent_config` / `skill` / `mcp_server` / `provider` / `plugin_market_package`）的 `visibility varchar(20) NOT NULL` 表达（前四张默认 `'private'`；`plugin_market_package` 默认 `'public'`，因为市场条目存在的意义就是被所有已认证用户看到，照抄 `private` 会让整个目录静默消失）；授权判断与查询谓词一律由 `@fenix/access-control` 产出，资源包只声明「资源类型 + 表 + 归属列 + 业务条件」。
  - `agent_site_app` 是第六个受控资源注册（`accessControlBindings`），但**它的 `visibility` 不是平台受众列**：该列是站点自身的发布范围（`private` / `org` / `authenticated` / `public`，决定谁能访问对外部署的站点），注册因此只声明归属列（`organizationId` / `ownerUserId`），发布范围的读过滤作为业务条件与平台谓词同条 SQL 下推。把它绑成平台受众列会同时破坏两端：`org` / `authenticated` 被按「非 public 即 private」处理（同组织成员读不到他人发布的站点），而 `public` 会给任意已认证用户开出跨组织读路径（站点行带 `platform_token`）。理由见 `packages/resources/agent-config/src/server/access/agent-site-app-resource.ts`。
- 提交迁移时必须提交完整 `drizzle/` 迁移链，不能遗漏 `drizzle/meta/*`。
- 禁止手写 SQL 迁移绕过 Drizzle，禁止在生产环境使用 `db:push`。
- 迁移设计必须考虑已有数据、锁范围、回滚或补偿策略，以及多实例并发启动时的幂等性。
- 生产迁移入口为 `scripts/migrate.ts` 构建出的 `migrate.js`。

