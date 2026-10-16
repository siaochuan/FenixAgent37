---
status: current
owner: <待指定>
verified-against: <commit>
source: 自原 CLAUDE.md v1.6 迁入（2026-10-16），内容未改动
---

# Skill、Agent 模板与 Permission

### Skill、Agent 模板与 Permission

- Skill 是 PostgreSQL 元数据、源目录和归档文件的组合存储；创建或导入必须通过 `setSkill` / `importSkillDirectories` 完成完整编排。
- 直接调用底层 `upsertSkill` 只更新元数据，不能替代文件写入、归档和失败回滚。
- Agent 模板位于 `.agents/agents/`，格式为 Markdown + YAML frontmatter，解析必须使用 `gray-matter`，不得手写正则。
- Permission action 只有 `ask` / `allow` / `deny`；规则型工具支持全局三态或 glob pattern 映射，开关型工具只支持三态字符串。

