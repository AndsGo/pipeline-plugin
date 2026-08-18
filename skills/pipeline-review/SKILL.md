---
name: pipeline-review
description: 流水线阶段⑤——独立代码评审。当工单实现完成（MR 已建或分支就绪）、需要以无实现上下文的独立视角评审 diff 时使用。调用参数为工单 ID，可选 base 提交（如 /pipeline-review TICKET-123 base=abc1234）。
disable-model-invocation: true
---

# Pipeline Review：独立代码评审

## 概述

以**独立、无实现上下文**的视角评审本工单的代码改动，产出双轴结论：**Spec 符合性**（逐条 AC）+ **代码质量**（分级 findings）。

**开始前必读**：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/handoff-spec.md`。

**参数**：工单 ID；可选 `base=<sha>`（diff 基点，编排器传入）。

## 独立性纪律（本 skill 的存在理由）

你的价值在于没有被实现过程"说服"过。因此：

- **评审依据只有三样**：`10-prd.md`（AC 清单与范围）、`20-plan.md` 的 **Global Constraints 一节**（只读这一节，不读任务详情——计划的具体写法不该锚定你对正确性的判断）、代码 diff。
- `05-knowledge-hints.md` 存在时可读：历史工单沉淀的踩坑提示，**只用作检查点提醒**（如"这类配置仓库里有三份要逐一核对"）。它不是评审依据——不得因提示放松或收紧对本次 diff 的判定，也不得把提示内容当成本单事实。
- **禁止读取**：`ledger.md`、`25-impl-report.md`、实现过程的任何记录。实现方声称"测试通过"不算证据，你自己跑。
- 评审中允许读仓库任意源码（理解上下文），允许运行测试和 AC 验证命令（独立复现）。

## Diff 获取

- 有 GitLab MR（glab 可用且 MR 存在）：`glab mr diff <mr>`。
- 否则用 git：编排器传了 `base=` 就用 `git diff <base>...HEAD`；没传则 `BLOCKED`（不要自己猜基点——猜错基点会把别人的提交审进来）。
- diff 为空 → `BLOCKED`（编排错误）。

## 评审过程

1. 读 AC 清单，为每条 AC 建立"我要在 diff/运行结果里看到什么"的预期（判据：每条 AC 都写下了预期证据形态，缺一条不得进入判定）。
2. 通读 diff（含足够的上下文行，必要时打开完整文件）。判据：diff 的每个 hunk 都被 Spec 轴与质量轴各检查过一次——按 hunk 清点，不凭印象。
3. **逐条 AC 判定**：`实现` / `未实现` / `diff 无法验证`（实现在未改动代码里、或需要运行环境）。自动 AC 亲自运行验证命令并记录输出；`diff 无法验证` 项不阻塞，但必须列出留给人工验收关注。
4. **质量扫描**，每条 finding 记录：severity（Critical / Important / Minor）、`file:line`、问题陈述、**失败场景**（什么输入/状态下产生什么错误后果——写不出失败场景的"问题"降级为 Minor 建议）。关注：正确性、边界条件、安全（本工单类型重点：密钥展示与存储、鉴权路径）、与 Global Constraints 的冲突、测试是否真的断言了行为。
5. **横切改动的全入口核查**：diff 触及共享的横切实现（鉴权/签发/校验、中间件、公共工具）时，按相关函数名做一次 repo-wide grep（如 CreateAccessToken / ParseJWT），**逐一列出所有消费同类凭据/能力的入口**并确认本次改动已覆盖或明确无需覆盖——只审 diff 里改到的调用点会漏掉没被改的消费方（LS-009 评审用此法确认全仓库仅两处消费开放平台 token 且均已接入新校验，固化为标准动作）。
6. 有 MR 时：Critical/Important findings 以行内评论落到 MR（discussions API / `glab api`）；无 MR 时只落文档。

## 结论映射

| 情形 | verdict | status |
|---|---|---|
| 全部 AC 实现（或仅"无法验证"项）且无 Critical/Important | `PASS` | `DONE` |
| 同上但有值得做的 Minor 建议 | `PASS_WITH_SUGGESTIONS` | `DONE_WITH_CONCERNS`（建议进 concerns） |
| 任何 AC 未实现，或存在 Critical/Important finding | `BLOCK` | `DONE`（评审本身完成了） |
| 输入缺失、diff 拿不到 | —— | `BLOCKED` |

**不要因为"总体不错"而软化结论**：一条 AC 未实现就是 BLOCK，让修复轮去解决——你的仁慈会在验收阶段变成业务方的惊讶。

## 产物

写 `docs/pipeline/<ticket-id>/30-review-r<N>.md`（N 取当前不存在的最小轮次号；frontmatter `stage: review`，加一行 `verdict: <结论>`，六章节齐全）。"本阶段结论"包含：

- AC 判定表（AC-n / 判定 / 证据：命令输出摘要或 diff 位置 `file:line`）
- Findings 清单（severity 降序，每条含 file:line 与失败场景）
- verdict 及一句话依据

"给下一阶段的指令"：BLOCK 时写给修复轮（哪些 findings 必须修、验证方式）；PASS 时写给验收阶段（哪些"无法验证"项需人工重点看）。

## 结束

按 handoff-spec §6 自查后输出结构化返回，含 `verdict` 与 **`axes` 双轴清点**（spec: AC 总数/未实现数/本轴最坏项；quality: Critical/Important/Minor 计数/本轴最坏项）——清点数字必须与文档里的 AC 判定表和 findings 清单逐一对得上。`summary_for_card` 面向业务方且**两轴分句各报一句**（Spec 轴一句、质量轴一句），不合并成单一总评。
