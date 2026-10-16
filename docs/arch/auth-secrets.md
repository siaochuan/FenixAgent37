---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# 认证、组织与密钥

### 认证、组织与密钥

- 普通 HTTP / WebSocket 请求先尝试 better-auth session cookie；无 session 时依次尝试 Environment Secret 和 better-auth API Key。
- 系统 API 使用独立的 `RCS_SYSTEM_API_KEYS`；`RCS_API_KEYS` 当前用于 skill 下载 token 的 HMAC 签名，不得混作普通请求认证规则。
- active organization 提取优先级：`x-active-org-id` header → `activeOrganizationId` query → `active_org_id` cookie。
- API Key 的组织上下文必须由 key metadata 恢复并重新校验成员关系；组织入口取自凭据，**角色取成员表当前值**，凭据里的创建期快照不参与判定；校验异常时保守拒绝。
- 测试可使用 `setTestAuth()`、`setTestOrgContext()` 注入上下文，测试结束必须 reset，避免状态泄漏。
- 密钥、token、密码和连接串不得写入源码、fixture、日志或错误响应。

