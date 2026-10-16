# 执行计划（AGENTS.md §2「执行计划入库」）

- 何时需要：跨模块、多步骤或预计超过 1 小时的任务。
- 位置：进行中 `active/<YYYY-MM-DD>-<slug>.md`；完成后 `git mv` 到 `completed/`。
- 多 agent 阶段交接（explore / plan / code / review）写在同一文件对应小节，替代临时 handoff 目录。

## 模板

```markdown
---
title:
owner:            # 唯一 owner（R-M1）
lanes: []         # 并行 lane 及各自写入范围
design: docs/design/<file>.md
status: active    # active | completed | abandoned
---

# <任务>

## 目标与范围（含明确不做的事）
## 计划阶段 6 问的回答
## 切片与里程碑
## Handoff
### 01-explore
### 02-plan（含 plan review 结论）
### 03-code（实际 diff 摘要与运行过的命令）
### 04-review（APPROVED 或问题清单）
## Progress（带时间戳，持续追加）
## Decision Log（含 Complexity Reset 记录，R-R2）
## Surprises & Discoveries
## Outcomes & Retrospective（可升级为 lint 的评审意见，R-E3）
```
