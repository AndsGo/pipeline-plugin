---
name: pipeline-implement
description: 流水线阶段④——实现/测试/文档。当工单的实现计划（20-plan.md）已审批通过、需要逐任务执行实现并产出 MR 时使用。调用参数为工单 ID（如 /pipeline-implement TICKET-123）。
disable-model-invocation: true
---

# Pipeline Implement：实现 / 测试 / 文档

## 概述

按 `20-plan.md` 逐任务执行实现。执行引擎复用 **superpowers:subagent-driven-development**（SDD：每任务派全新子代理 + 任务级评审 + 修复回环上限 + 全分支终审），本 skill 只定义流水线适配规则；SDD 的过程纪律（模型分层、评审包、fix loop、breaker 仲裁）全部照其原文执行。

**开始前必读**：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/handoff-spec.md`，然后加载 superpowers:subagent-driven-development。

**参数**：工单 ID。**输入**：`docs/pipeline/<ticket-id>/20-plan.md`（frontmatter 须为 `stage: plan` 且 `status: DONE` 或 `DONE_WITH_CONCERNS`，否则 `BLOCKED`）；`05-knowledge-hints.md` 存在时先读——历史工单的踩坑提示（环境限制、配置陷阱），执行中主动规避，与计划冲突时以计划为准。审批是否通过由编排器保证——本 skill 被调起即视为已获批，不重复校验。

## 对 SDD 的三条覆盖规则（与其原文冲突时以本节为准）

1. **台账位置**：进度台账固定为 `docs/pipeline/<ticket-id>/ledger.md`（不用 SDD 默认的 `.superpowers/sdd/` git-ignored 目录）。首行身份 `# SDD ledger — plan: docs/pipeline/<ticket-id>/20-plan.md`。**每完成一个任务，ledger 的更新与该任务的代码一起 commit**——这是流水线断点恢复的依据：本 skill 重新被调起时，先读 ledger，带 `Task N: complete` 行的任务一律不重派，从第一个没有完成行的任务继续。
2. **无人可问**：SDD 中所有"询问人类伙伴"的分支（计划冲突仲裁、BLOCKED 上报）改为结束会话并结构化返回：计划文本间冲突或计划与评审规则冲突 → `BLOCKED`（`blocked_reason` 附冲突双方原文）；缺业务信息 → `NEEDS_CONTEXT` + open_questions。**不要原地等待，不要自行替业务方裁决。**
3. **产物与出口**：全部任务完成且终审干净后——
   - 跑计划里声明的自动 AC 验证命令，结果记入实现报告（判据：计划声明的每条自动 AC 都有「命令 + 输出摘要」逐条对应，缺一条即本阶段未完成）；
   - 有 GitLab 远端且 glab 可用 → `glab mr create`（标题含工单 ID，描述链接 10-prd.md 与 20-plan.md）；无远端或 glab 未配置（试跑环境）→ 跳过 MR，作为 concern 记录；
   - 写 `docs/pipeline/<ticket-id>/25-impl-report.md`（frontmatter `stage: implement`，六章节齐全），"本阶段结论"含：任务完成清单、测试总况、自动 AC 验证结果、MR 链接（或未创建原因）、parked findings 汇总。

## 分支与工作区

- 开工第一步：确认不在 main/master 上。分支名按计划 Global Constraints（`feat/<ticket-id>-<slug>`）；分支已存在（断点恢复场景）则直接切换续用。
- 建分支后先把 `docs/pipeline/<ticket-id>/` 下已有工件提交一笔（`chore(pipeline): LS-xxx intake/prd/plan`），保证工件随分支走。
- 仓库有本工单之外的未提交改动 → `BLOCKED`（不要替别人的工作现场做决定）。

**人工反馈**：参数含 `feedback=<path>` 时按 handoff-spec §8 先读，人的指令优先于计划里的旧写法；因反馈偏离计划的地方在 ledger 与实现报告里写明依据。

## 修复轮（review 打回时的二次进入）

编排器带 findings 文件调起（如 `/pipeline-implement TICKET-123 fix=docs/pipeline/TICKET-123/30-review-r1.md`）时，进入修复模式：

1. 读 findings 文件与 ledger，**只处理 verdict 为 BLOCK 的 findings**（Critical/Important 与未实现的 AC）；Minor 建议不在修复轮范围。
2. 每条 finding 按 SDD 修复回环执行：派实现子代理修复 → 跑覆盖测试 → scoped 复审。ledger 追加 `Review fix round: <finding 摘要> (commits ...)`。
3. 完成后在 `25-impl-report.md` 追加"修复记录"段（不改写原报告），逐条列 finding → 处置（已修复/有异议）。**对 finding 有异议时不得静默丢弃**：写明理由，进 concerns，由编排器交人工仲裁。
4. 状态出口同下表；全部目标 findings 处理完即 `DONE`（或带异议的 `DONE_WITH_CONCERNS`）。

## 状态出口速查

| 情形 | 状态 |
|---|---|
| 全部任务完成、终审干净、自动 AC 通过、MR 已建 | `DONE` |
| 完成但有 parked findings / 无法建 MR / 个别自动 AC 缺少运行环境 | `DONE_WITH_CONCERNS`（逐条列入 concerns） |
| 计划缺业务决策信息 | `NEEDS_CONTEXT`（带推荐答案） |
| 计划缺陷、fix loop 到顶且 load-bearing、环境不可用（依赖装不上、测试基础设施坏） | `BLOCKED` |

## 结束

按 handoff-spec §6 自查后输出结构化返回，`handoff_path` 指向 `25-impl-report.md`。`summary_for_card` 面向业务方：完成了什么、测试结论、下一步是 review。
