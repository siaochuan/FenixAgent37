---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6「质量、测试与长期维护」迁入（2026-10-16），质量红线已上收到 AGENTS.md R-G1/R-G2
---

# 测试、代码注释与命名约定

### 测试

- 后端测试位于 `apps/server/src/__tests__/`，前端测试位于 `apps/web/src/__tests__/`。
- 优先复用 `apps/server/src/test-utils/`，测试文件禁止直接调用 `mock.module()`。
- 每个 `test(...)` 上方添加一行中文注释，说明行为和业务意图。
- 前端只测试关键交互、状态和数据流，不编写纯 UI 结构断言或仅重复类型检查的测试。
- 并发、重连、权限、租户隔离、迁移和失败回滚必须覆盖关键边界测试。

### 代码与注释

- TypeScript 业务代码禁止 `as any`；确因第三方类型缺陷需要规避时，使用最小范围的类型收窄或带原因的行级 ignore。
- Zod 使用 `zod/v4`。
- 允许空字符串的默认值使用 `??`，不得使用会吞掉空字符串的 `||`。
- `catch` 必须保留诊断上下文，不得吞错；对外错误不得泄露敏感信息或内部实现。
- 公共函数、公共方法、导出工具和类型定义应有清晰文档注释。
- 注释解释设计原因、边界条件、兼容约束、风险和临时取舍，不重复代码表面行为。
- 已知缺陷或临时方案必须记录影响范围、风险、移除条件和可追踪任务；关键架构变更同步其仍有效的事实与约束到对应架构文档。

### 命名与 Git

- 文件使用 kebab-case，组件使用 PascalCase，函数使用 camelCase，常量使用 UPPER_SNAKE_CASE。
- 提交信息使用 Angular 风格：`feat:` / `fix:` / `refactor:` / `test:` / `chore:` / `docs:`，标题使用中文。
- 未经明确要求不得创建 commit；代码改动提交前通常须运行 `bun run precheck`，前端改动还须运行 `bun run build:web`。
