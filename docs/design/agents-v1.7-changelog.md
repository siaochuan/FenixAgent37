# CHANGELOG：CLAUDE.md v1.6 → AGENTS.md v1.7（2026-10-16，草案，未推送）

来源缩写：H = OpenAI《Harness engineering》；P = PostHog AGENTS.md；O = openai-agents-python AGENTS.md/PLANS.md；I = Fenix 内部不一致（见 fenix-vs-harness-20261016.md 第二节）。
"可强制"：是 = 可完全由门禁实现；部分 = 结构可校验、内容靠评审；否 = 仅评审。标"待建"的门禁需新写脚本。

| # | 变更 | 原因 | 来源 | 可强制 |
|---|---|---|---|---|
| 1 | 改为 AGENTS.md 唯一事实源，CLAUDE.md 只做指针 | 多厂商 agent（Codex/Claude/Grok/GLM）读不同文件名，避免多份漂移 | agents.md 惯例 / I | 是（可检查 CLAUDE.md 不含规则） |
| 2 | 入口从 264 行瘦身到约 115 行，11 组领域细则原文迁入 `docs/arch/*.md`，入口保留"改动区域 → 必读"地图 | "give Codex a map, not a 1,000-page instruction manual" | H | 部分（入口长度 lint；必读靠评审） |
| 3 | 每条规则加 ID 与 `[lint:xxx]` / `[review]` 标注，未实现门禁标"待建" | 让 agent 区分机器拦截与自觉遵守，暴露未工具化规则 | P | 是（规则 ID 唯一性检查） |
| 4 | 新增 R-00 规则权威顺序，单一定义只引用不复述 | 原文同时以 engineering-standards 与"离代码更近者"为准，两个最高权威 | I | 部分（重复段落检测） |
| 5 | R-A2 明确"设计完整 vs 第二用例才抽象"判别：设计覆盖边界/契约/失败路径，代码抽象只服务已存在用例 | 原条目两句并列无判别标准 | I | 否 |
| 6 | R-A3 500 行由 `check:file-size` 强制 + 只减不增台账 | 原规则无执行者，实测约 40 个文件超限 | H（file size limits 用 lint）/ I | 是（待建） |
| 7 | R-A7 异常分层：内部原样抛，协议边界转换，原始错误进日志 | "不可替换"与"对外不得泄露"字面冲突 | I | 部分（空 catch / catch 不重抛不记日志 lint，待建） |
| 8 | R-R1 删除优于兼容补"适用前提"与对外契约清单（含 DB 两阶段删除、落库 ID、plugin-sdk、cross-repo 接口） | 原规则依赖未写明的单仓库+全量测试前提；engineering-debt 已有事实例外 | I | 部分 |
| 9 | 新增 R-R2 Complexity Reset | 防止同一设计不断叠加兼容分支 | O | 否（Decision Log 记录可检查存在性） |
| 10 | 新增执行计划入库 `docs/exec-plans/`，auto-devflow handoff 改写入计划文件 | 原 handoff 为"temporary"，与"为未来维护者保留上下文"矛盾；计划作为一等产物 | H / O | 部分（待建） |
| 11 | R-G3 门禁报错必须含规则 ID、文档链接、修复步骤 | "write the error messages to inject remediation instructions into agent context" | H | 是（待建，给门禁脚本加测试） |
| 12 | R-G4 flaky 通道：重跑一次 + 登记，禁止删断言/放宽超时/skip | 保留全绿门禁同时避免 agent 为过门禁改坏测试；比 Harness"多重跑"更保守 | H（反向取舍）/ I | 部分（待建） |
| 13 | §4 异步裁决请求（`type: adjudication` + `approved-by`）+ 低风险事后报备 + 不阻塞并行 | 人类审批与多 agent 并行冲突；审批颗粒度与记录方式未定义 | I / P（"批准不可推断"精神） | 部分（迁移/依赖 PR 链接校验，待建） |
| 14 | R-M2 能力层（检索/实现/架构）替代 haiku/sonnet/opus | multitask skill 写死单一厂商型号与 crate 术语 | I | 否 |
| 15 | R-M3 并行 lane 合流后由集成 owner 跑一次 precheck | 原规则未说明并行合流时谁负责全量门禁 | I | 部分 |
| 16 | §6 每周清理/文档巡检 agent（台账到期、超限、TODO、死链、arch 过期、design 未归档） | "doc-gardening" agent 与"garbage collection" | H | 是（定时任务，待建） |
| 17 | R-E2 design frontmatter + 生命周期 + 归档流程；arch 加 `verified-against` | 实测 5/14 篇缺状态、2 篇已实现仍留在 design | H / I | 是（待建） |
| 18 | R-E3 同类评审意见第二次出现即提议 lint | "When documentation falls short, we promote the rule into code" | H | 否 |
| 19 | R-D2 允许对不透明依赖提议自写最小实现（仍需裁决） | 吸收 Harness 取向但保留 Fenix 审批 | H | 否 |
| 20 | 新增台账 `docs/ledgers/file-size-exceptions.md`、`flaky-tests.md`；模板 `docs/design/_template.md`、`docs/exec-plans/README.md` | 支撑 6/10/12/17 | — | — |

## 刻意保留（未向 Harness 靠拢）
1. precheck 全绿 + warning 清零 + 不得以"历史问题"宣称通过：低吞吐、多租户企业场景下优于 Harness 的"minimal blocking merge gates"。
2. 表结构 / 新依赖 / 对外契约须人类裁决：数据不可逆；仅改为异步结构化。
3. 架构例外台账（禁通配、removeWhen、阶段末清空）原样保留。

## 未包含 / 待办
1. 所有"待建"门禁脚本未实现；落地前按 `[review]` 执行。
2. `docs/arch/*.md` 迁入内容逐字保留原文，未与现有 docs/arch 同名篇章合并（如 `19-yjs-chat-streaming.md`），合并需人工判断。
3. engineering-standards.md 未修订，需按 R-00 去除与本文件重复的段落。
4. multitask / auto-devflow skill 文件本身未改写，仅在规范层给出约束。
