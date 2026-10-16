# docs/design 使用说明与生命周期

## Frontmatter 模板（每篇必填，`check:docs-frontmatter` 校验，待建）

```yaml
---
title: <标题>
type: design            # design | adjudication（裁决请求）| issue
status: draft           # draft → approved → frozen → implemented → (archived) ；或 superseded / rejected
owner: <人或 agent lane>
created: 2026-10-16
updated: 2026-10-16
cross-repo: false       # true 表示接口跨仓，适用 AGENTS.md R-R1 对外契约
exec-plan: docs/exec-plans/active/<file>.md   # 可选
supersedes: []          # 可选
approved-by: ""         # type: adjudication 必填后方可实施，如 "小川 2026-10-16"
---
```

裁决请求（`type: adjudication`）正文必含：背景 / 选项（≥2）/ 推荐 / 影响（数据、契约、性能、安全）/ 回滚方案。

## 状态流转

1. `draft`：起草，可随意修改。
2. `approved`：评审或裁决通过，可开工。
3. `frozen`：作为并行实现的接口契约，偏离必须先改本文件并说明原因。
4. `implemented`：已落地。**同一 PR 内**完成归档：
   1. 把落地后仍有效的事实、契约与约束合并进 `docs/arch/<对应篇>.md`，更新其 `verified-against`；
   2. 将本文 `git mv` 到 `docs/design/archive/`，并在 frontmatter 加 `archived: <date>` 与 `merged-into: docs/arch/<file>`；
   3. 删除过程性内容不需要搬运（讨论、备选方案保留在归档文件里即可）。
5. `superseded` / `rejected`：写明替代文档或理由，直接移入 archive。

## 巡检（AGENTS.md R-E1）

每周清理任务报告：缺 frontmatter、`implemented` 但未归档、`approved` 超过 60 天无对应执行计划、`cross-repo: true` 但无契约快照的文档。
