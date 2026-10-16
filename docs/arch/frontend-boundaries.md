---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# 前端边界与体验

### 前端边界与体验

- 导航只使用 `<Link to>` 和 `useNavigate()`；禁止 `window.location.href`、`window.location.replace`、`window.location.reload` 和 `window.history.pushState`。Sidebar 导航项由各包 contribution 声明 `id`（`id` 即路由目标，Shell 拼成 `/agent/<id>`），没有 `to` 字段（见前端规范 §2.6）。
- 请求统一通过 `@fenix/web-runtime/api/request`（真身 `packages/web-runtime/web/api/request.ts`）；`request<T>()` 处理路径参数、query、JSON、超时与错误标准化，但**返回 `ApiResponse` 而非已解包数据**——失败时返回 `{ success: false }` 而不 throw，调用方必须 `unwrap()` 或显式判断 `success`；直接 `await` 并依赖 `catch`/`onError` 会把 4xx/5xx 当成成功（目标为单一异常语义，登记见 `docs/developer/architecture/engineering-standards.md` §11「优化项」）。
- 数据获取优先遵循前端规范和现有 `ahooks` / `useRequest` 模式，避免重复请求与竞态覆盖。
- 用户可见字符串必须通过 `t()`；i18n 插值使用 `{{var}}`，单花括号 `{var}` 会被当作字面文本。
- 基础组件优先复用 `@fenix/ui-components/ui/<name>`（该包无根出口，逐文件子路径）；通用图标使用 `lucide-react`；模型品牌图标使用 `packages/resources/model-management/web/components/model-icon/ModelIcon.tsx`。
- 纯逻辑模块不得依赖 UI 图标包；特别是不得让后端或纯逻辑测试间接加载 `@lobehub/icons`。
- 页面流程必须覆盖 loading、empty、error、retry、success feedback 和可访问性状态。
- 路径别名只保留宿主自有条目（`@/src/*` → `apps/web/src/*`、`@server/*` → `apps/server/src/*`，含 `@/src/i18n`、`@/src/i18n/locales`、`@/src/lib/random-uuid-polyfill` 等细粒度项）；跨包引用一律经各包 `exports`，禁止再新增指向 `packages/**` 的别名。`apps/web/vite.config.ts` 与根 `tsconfig.json` 两张表必须逐条一致（dependency-cruiser 读根表）。

