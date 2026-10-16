---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# API 与数据模型边界

### 后端分层

新增与重写后端代码以 `docs/developer/system/backend-development.md` 为准：外部动作经 Resource Facade 完成授权、状态校验与编排；Domain Service 不接收 actor；Repository 只封装持久化与事务原语；route 仅作协议适配且不直接访问 db。资源 schema 与 route 归各自 owner 包，宿主仅聚合协议入口。

### API 与数据模型边界

- `/web/*` 服务控制台前端，默认响应 `{ success, data }` 或 `{ success: false, error }`。
- `/api/*` 服务外部系统和 API Key 调用方，优先保证向后兼容。
- ACP、MCP、hooks、skills 等内部协议能力使用独立前缀，不得混入 `/web` 或 `/api`。
- 新接口默认补齐 OpenAPI 的 `detail`、`params`、`query`、`headers`、`body` 和 `response`。
- 协议 DTO、领域对象、数据库记录和前端 ViewModel 在边界处独立转换；前端类型必须对应后端真实返回，禁止增加“幻影字段”。
- 前端 URL 统一使用 `/web/*`，不得新增历史 `/v1`、`/v2` 前缀。
- `POST /web/config/:module` 保持既有 action 风格；同一路由文件可能混用 RESTful 与 action 风格，修改前必须核对实际协议，不得擅自统一。

