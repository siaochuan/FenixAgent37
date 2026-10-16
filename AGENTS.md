# AGENTS.md（FenixAgent 工程规范 v1.7 · 唯一事实源）

> 本文件是所有 agent（无论哪家厂商）的唯一入口；`CLAUDE.md` 仅指向本文件。每条规则带 ID 与执行方式：`[lint:检查名]` 由门禁强制，`[review]` 由评审把关，标注"（待建）"的门禁尚未实现，落地前按 `[review]` 执行。
> 汇报风格：用户有阅读障碍，减少汇报字数，多用 order list 结构化表达。

## 0. 规则权威顺序

1. **R-00 权威顺序** `[review]`：对外协议契约 > 本文件 > `docs/developer/architecture/engineering-standards.md` 细则 > `docs/arch/` 现行事实 > 包内 README。同一规则只在一处定义，其他文档只引用不复述；文档与代码不一致时先核实设计意图并同步修正，不得静默沿用冲突规则。

## 1. 核心工程原则

架构设计

1. **R-A1 架构与领域优先** `[review]`：计划阶段以理想架构为目标，明确业务目标、领域边界、模块职责、依赖方向和数据流后再编码；不得以短期便利牺牲整体设计。
2. **R-A2 设计完整、实现克制** `[review]`：判别标准——*设计*必须覆盖边界、契约、数据流、失败路径与演进方向（写在 plan/design 里）；*代码抽象*只为已存在的用例服务，第二个真实用例出现时才提取公共接口，单一场景直接实现。设计里可以预留扩展点，代码里不预建。
3. **R-A3 文件规模** `[lint:check:file-size]`（待建）：单个源文件（不含测试与生成文件）不得超过 500 行，接近上限时优先重构模块边界；存量超限文件登记于 `docs/ledgers/file-size-exceptions.md`（owner + removeWhen），台账只减不增。
4. **R-A4 业务规则单一权威** `[review]`：协议模型、领域模型、持久化模型和视图模型不得相互泄漏；数据在边界处校验并独立转换，避免跨层共享可变状态。
5. **R-A5 interface + adapter** `[review]`：同一领域规则只维护一份 interface，多个 adapter 封装差异。
6. **R-A6 依赖方向由门禁强制** `[lint:check:dependencies]` `[lint:architecture:check]`：类别间允许方向见 engineering-standards §2.3 依赖矩阵；跨包只经公开 export。存量违规只能登记在架构例外台账（`scripts/lib/architecture-exceptions.ts`），条目须带 owner、removeWhen、rationale，禁止通配，阶段末清空。
7. **R-A7 异常分层** `[lint:no-swallowed-catch]`（待建）：领域内部原样抛出原始错误，不吞、不替换、不降级为默认值；只在协议边界（route、ACP translator、外部 API）把错误映射为对外错误码，映射时原始错误与诊断上下文进日志，对外响应不得泄露敏感信息或内部实现。

开发

1. **R-D1 面向并发与故障设计** `[review]`：主动考虑幂等、竞态、事务边界、超时、取消、重试、背压和资源释放；不得用无边界重试、吞错或隐式共享状态掩盖问题。
2. **R-D2 复用优先** `[review]`：严格复用已有组件；新增模块或依赖须走裁决请求（§4）。对不透明、难以测试或与可观测性冲突的依赖，可在裁决请求中提议自写最小实现。
3. **R-D3 先验证完整行为** `[review]`：先验证用户可观察行为，再按风险补回归、失败和生命周期测试；前端流程覆盖加载、空、错误、重试、反馈和可访问性。
4. **R-D4 克制地保留上下文** `[lint:check:todo-context]`（待建）：非显然决策、兼容约束、已知缺陷和临时方案须记录原因、影响、风险和移除条件，并关联可追踪任务；禁止缺少上下文的 `TODO`。文风克制，一般 2 句内写完。
5. **R-D5 安全与隔离默认开启** `[review]`：按多租户、多用户设计，最小权限，外部输入不可信，敏感信息不得进入代码、日志或响应。

测试

1. **R-T1 先局部后全局** `[review]`：快速验证为先，集成测试为后。
2. **R-T2 可验证、可观测、可回滚** `[review]`：每项改动行为可测、状态可观测、故障可定位，兼顾向后兼容与回滚路径。
3. **R-T3 修复一类问题** `[review]`：先找同类原因的触发位置，手术刀式修复；用户反馈的问题留存到 `docs/design/issues/`。

重构

1. **R-R1 删除优于兼容** `[review]`：内部路径重构时直接删除过时实现，禁止新增兼容层、deprecated shim 或双写逻辑。
   - **适用前提**：被删符号仅被本仓库代码引用；不落库、不跨进程、不跨仓；`precheck` 全绿证明无残留调用方。任一前提不满足即按对外契约处理。
   - **对外契约清单**（不适用直接删除，按契约单独评估）：`/api/*` 路由；`@fenix/plugin-sdk` 及其他对外发布包的导出；ACP / MCP / hooks / skills 协议格式；已落库的 ID、枚举值与数据迁移 ID；DB 表与列（按"停写 → 停读 → 下一发布 DROP"两阶段删除）；`docs/design/` 中标记 `cross-repo: true` 的接口。
2. **R-R2 Complexity Reset** `[review]`：同一设计第二次因相关发现需要再加兼容分支、状态字段或协议跳转时，必须停止追加条件，归并根因，对照原始需求重审完整 diff，并在执行计划的 Decision Log 记录；优先删除不受支持的机制或拒绝不受支持的输入。

## 2. 开发工作流

### 计划阶段（编码前必答 6 问）`[review]`

1. 确认需求、领域术语、模块边界、数据流和向后兼容要求。
2. 查找已有实现、公共接口和相邻模块，优先深化现有模块，不并行创建第二套能力。
3. 明确多租户隔离、权限校验、并发与失败路径。
4. 定义验证方式、可观测信号和回滚路径。
5. 长期架构变更同步规划到 `docs/arch/`；未完成或待评审方案放入 `docs/design/`（模板与生命周期见 `docs/design/README.md`）。
6. 大功能按可运行的垂直切片分阶段交付，每层从最小端到端版本起步，不一次性铺完整层再联调。

### 执行计划入库 `[lint:check:exec-plans]`（待建）

跨模块、多步骤或预计超过 1 小时的任务，计划写入 `docs/exec-plans/active/<YYYY-MM-DD>-<slug>.md`（模板见该目录 README），执行中持续更新 Progress / Decision Log / Surprises；完成后移入 `completed/`。多 agent 阶段交接（explore → plan → code → review）的 handoff 写在同一计划文件中，不使用临时目录。

### 任务范围 `[review]`

1. 变更只覆盖已确认目标；不提前实现后续阶段，不夹带无关重构、框架或通用抽象。每条变更可追溯到需求、架构约束、缺陷修复或必要验证。

## 3. 门禁

| 命令 | 时机 | 说明 |
| --- | --- | --- |
| `bun run fastcheck` | 单任务收尾 | 静态门禁 + 受影响测试；不替代 precheck |
| `bun run precheck` | 提交前 | 全量门禁，实际步骤以 `scripts/ci.ts` 为准 |
| `bun run build:web` | 改前端后 | 后端从 `apps/web/dist/` 挂载，不可省略 |
| `bun run db:generate --name <module>-<change>` → `db:migrate` → `run-data-migrations` | 改 schema 后 | 存量数据搬迁时才跑最后一步 |
| `bun run docs:build` | 改文档站点后 | — |

1. **R-G1 全绿** `[lint:precheck]`：常规任务提交前 `precheck` 必须全绿；不得新增 typecheck、lint 或测试错误，linter warning 清零（info 除外）。
2. **R-G2 不得伪造通过** `[review]`：`precheck` 失败必须处理后才能宣称完成；失败若来自与本改动无关且无法安全修复的工作区状态，须报告证据和影响，不得以"历史问题"为由宣称通过。自动修复只用于已审查范围，不得为通过检查而改写无关文件。
3. **R-G3 门禁报错可修复** `[lint:check:gate-messages]`（待建）：所有自研门禁失败信息必须包含规则 ID、规则文档链接、修复步骤或例外登记方式。
4. **R-G4 Flaky 通道** `[lint:check:flaky-ledger]`（待建）：疑似不稳定的测试允许重跑一次；仍失败按真实失败处理。确认不稳定的测试登记到 `docs/ledgers/flaky-tests.md` 并关联修复任务；禁止删除断言、放宽超时或加 skip 来过门禁。
5. **R-G5 TypeScript 红线** `[lint:biome]`：业务代码禁止 `as any`（第三方类型缺陷用最小范围收窄或带原因的行级 ignore）；Zod 用 `zod/v4`；允许空字符串的默认值用 `??`。

## 4. 人类裁决（异步）

1. **R-H1 需裁决事项** `[lint:check:adjudication-link]`（待建）：数据库表结构变更、新增模块或依赖、对外契约变更，以"裁决请求"提交——一份 `docs/design/` 文档（`type: adjudication`），写明选项、推荐、影响与回滚；用户在 frontmatter 填 `approved-by` 与日期后方可实施，迁移或依赖 PR 必须链接该文档。
2. **R-H2 低风险事后报备** `[review]`：新增可空列、新增索引、patch 级依赖升级、内部重命名，可先实施，在 PR 描述"事后报备"段列出，用户可追溯否决。
3. **R-H3 不阻塞并行**：等待裁决的 lane 挂起本 lane，其他 lane 继续；不得以"等待裁决"为由停下全部工作。
4. **R-H4 提交** `[review]`：未经明确要求不得创建 commit；提交信息用 Angular 风格（`feat:` / `fix:` / `refactor:` / `test:` / `chore:` / `docs:`），标题中文。

## 5. 多 agent 协作

1. **R-M1 一个工作流一个 owner** `[review]`：每个交付物（调查 → 实现 → 验证）只有一个 owner；并行 owner 写入范围不得重叠，接口先冻结。协调者不亲自改文件。
2. **R-M2 能力层而非型号**：派发按能力层表述——*检索层*（查找、机械扫描）、*实现层*（带验证的实现）、*架构层*（跨包契约与架构取舍）。层到具体模型/执行器的映射放在执行器配置中，不写进规范。
3. **R-M3 合流** `[lint:precheck]`：并行 lane 合流后由集成 owner 在合流结果上跑一次 `precheck`，各 lane 自身的 fastcheck 不能替代。

## 6. 熵治理与文档新鲜度

1. **R-E1 每周清理任务** `[lint:weekly-gardening]`（待建）：定时 agent 扫描并开小 PR（单个不超过 300 行，可一分钟审完）：架构例外与文件规模台账到期项、超限文件、无上下文 TODO、flaky 台账、`docs/` 失效路径与死链、`verified-against` 超过 30 天的 arch 文档、状态为 implemented 却仍在 design 的文档。
2. **R-E2 文档生命周期** `[lint:check:docs-frontmatter]`（待建）：`docs/design/` 每篇必须带 frontmatter（模板见 `docs/design/README.md`）；status 变为 implemented 的同一 PR 内，把仍有效的事实迁入 `docs/arch/`，原文移入 `docs/design/archive/`。`docs/arch/` 每篇声明 `verified-against: <commit>`。
3. **R-E3 评审意见升级为工具** `[review]`：同一类评审意见出现第二次时，应提议对应 lint 或结构测试，并登记到执行计划或工程债务。

## 7. 文档地图（改动相关区域前必读）

| 改动区域 | 必读 |
| --- | --- |
| 目录、包分类、前后端地图 | `docs/arch/project-map.md` |
| `/web` `/api` 路由、DTO、OpenAPI | `docs/arch/api-boundaries.md` |
| 前端导航、请求、i18n、组件 | `docs/arch/frontend-boundaries.md`；`docs/developer/system/frontend-development.md` |
| 后端分层（route → Facade → Domain Service / Repository） | `docs/developer/system/backend-development.md` |
| HTTP / Workflow / Chat 调 Agent | `docs/arch/agent-communication.md` |
| 认证、组织、API Key、密钥 | `docs/arch/auth-secrets.md` |
| workspace 路径、远程文件 API | `docs/arch/workspace-files.md` |
| Skill、Agent 模板、Permission | `docs/arch/skill-template-permission.md` |
| acp-link、relay、ID 体系 | `docs/arch/acp-runtime.md` |
| YJS、Chat 状态、多标签页 | `docs/arch/yjs-chat-invariants.md`（十一条不变量，改动前逐条核对） |
| schema、迁移、可见性 | `docs/arch/database-migrations.md` |
| 环境变量 | `docs/arch/env-vars.md` |
| 测试、注释、命名、Git 约定 | `docs/arch/quality-conventions.md` |
