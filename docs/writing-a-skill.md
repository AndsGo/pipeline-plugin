# 写一个 skill

一个阶段 skill 回答四个问题：**读什么、做什么、做到什么程度算完、怎么返回**。本文按这个顺序讲结构与纪律，最后说怎么给它配评估。

## 1. 文件结构

```
skills/pipeline-<stage>/
└── SKILL.md
```

frontmatter：

```yaml
---
name: pipeline-review
description: 流水线阶段⑤——独立代码评审。当工单实现完成、需要以无实现上下文的独立视角评审 diff 时使用。调用参数为工单 ID，可选 base 提供 diff 基点。
disable-model-invocation: true    # 阶段 skill 只由编排器调起
---
```

`description` 要写清**什么时候用**与**参数是什么**——它是 Claude Code 决定要不要加载这个 skill 的依据。

## 2. 开头两句固定写法

```markdown
**开始前必读**：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/handoff-spec.md`。

**参数**：工单 ID。**输入**：`docs/pipeline/<ticket-id>/10-prd.md`（frontmatter 须为 …）、…；
`05-knowledge-hints.md` 存在时先读；`07-project-profile.md` 存在时**先读它**——项目事实优先于通用流程。输入缺失 → `BLOCKED`。
```

输入检查放最前面：缺输入直接 `BLOCKED`，不猜。

## 3. 正文：动作 + 完成判据

每一步都要能被判定"做完了没有"。对比：

| 不合格 | 合格 |
|---|---|
| 充分评审 diff | 每个 hunk 被 Spec 轴与 Quality 轴各查一次，结果进两张表 |
| 补充边界情况 | 对 AC 清单里标注「自动」的每一条，运行其验证命令并记录退出码与关键输出 |
| 适当处理 findings | 每条 finding 带 `file:line` 与一个具体的失败场景；写不出失败场景的降为 Minor |

需要划边界时，用**禁止读取**这类明确的负向表达（评审的独立性靠"禁止读 ledger.md 与实现报告"来保证），其余尽量说目标行为。

## 4. 产物：机器头 + 六节人类体

交接文档的形状由 handoff-spec §2 规定，每个 skill 照抄：

```markdown
---
ticket: <id>
stage: <stage>
status: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
produced: <ISO 8601 带时区>
inputs: [本阶段读了什么]
next_stage_reads: [下游必读]
open_questions: []      # NEEDS_CONTEXT 时非空
---
## 本阶段结论
## 关键决策及理由        （含放弃的备选与理由）
## 事实 vs 假设          （分区写；无假设写"无假设"）
## 失败过的尝试          （下游不得原样重提）
## 阻塞与开放问题
## 给下一阶段的指令      （零上下文可执行）
```

六节必须全部存在，无内容写"无"。**「事实 vs 假设」是最该照抄的一节**——它强迫会话把"查证过的"和"推测的"分开，下游据此知道哪些能直接用。

## 5. 结论映射：查表，不临场判断

review 与 acceptance 这类要给 verdict 的 skill，把"什么情形对应什么结论"写成表：

| 情形 | verdict | status |
|---|---|---|
| 全部 AC 实现且无 Critical/Important | `PASS` | `DONE` |
| 同上但有 Minor 建议 | `PASS_WITH_SUGGESTIONS` | `DONE_WITH_CONCERNS` |
| 任何 AC 未实现，或有 Critical/Important | `BLOCK` | `DONE`（评审本身完成了） |
| 输入缺失 | — | `BLOCKED` |

并在表下明说不许软化：「一条 AC 未实现就是 BLOCK——你的仁慈会在验收阶段变成业务方的惊讶」。

## 6. 结束：自查再返回

```markdown
## 结束

按 handoff-spec §6 自查后输出结构化返回，`handoff_path` 指向本阶段交接文档，
`summary_for_card` 面向业务方：完成了什么、结论是什么、需要谁做什么。
```

自查清单（§6）五条：自包含、假设诚实、frontmatter 与结构化返回一致、六节完整、status 与正文相符。

派了子代理的 skill 还要加一条（0.12.1 起）：**返回前用 `git log` / `git status` 核对每个子代理都已结束且已提交**，子代理还在后台跑就不返回；万一必须中断，返回里显式标「运行中快照，非最终状态」。

## 7. "问人"的唯一写法

流水线会话无人值守。任何"停下来问一下"都写成：

```yaml
status: NEEDS_CONTEXT
open_questions:
  - id: Q1
    question: "库存同步失败时应重试还是直接标记下架？"
    options: ["重试 3 次后标记下架", "立即标记下架"]   # ≥2 项
    recommended: "重试 3 次后标记下架"
    why: "现有 sync 模块已内置指数退避重试，复用成本最低"
```

每题必带推荐答案与理由——给不出推荐说明调查没做完，先调查再问。编排器把它渲染成卡片，答完重跑本阶段，上一轮问答已追加进输入文件。散文提问等于回程校验失败。

## 8. 项目约定的注入

编排器会把目标仓库 `docs/pipeline/PIPELINE.md` 里「全阶段 + 本阶段」两节抄进 `07-project-profile.md`。skill 开头必须写「存在时先读它，与通用流程冲突时以它为准」。这是把"这个仓库不能全新安装""这个项目没有测试环境"这类只有项目组知道的事告诉会话的通道。

## 9. 给它配一个评估场景

改完 skill 必须跑评估集。新场景放 `evals/<场景名>/`：

```
evals/review-blocks-missed-ac/
├── expect.json           # 断言
├── shared/               # 工作区文件（工件、源码）
│   └── docs/pipeline/T-1/{10-prd.md, 20-plan.md}
├── base/src/calc.js      # 可选：构造 diff 的基线
└── head/src/calc.js      # 可选：构造 diff 的改动后
```

```json
{
  "skill": "review",
  "why": "两条 AC 只实现一条——评审必须 BLOCK 且 Spec 轴清点出未实现项（不得因『总体不错』软化）",
  "extraArgs": "base={BASE_SHA}",
  "assert": { "status": ["DONE", "DONE_WITH_CONCERNS"], "verdict": "BLOCK", "axes_spec_failed_min": 1 }
}
```

`why` 写清这个场景守的是哪条纪律、来自哪个事故。跑法见 [`evals/README.md`](../evals/README.md)。

## 10. 三条真机教训

- **纪律靠"禁止读什么"保证，不靠"请客观"。** 评审独立性写成"禁止读取 ledger.md、25-impl-report.md、实现过程的任何记录；实现方声称测试通过不算证据，你自己跑"。
- **独立性也要切得准。** LS-012：r3/r4 确认的 Critical 从未修复，r5 的独立分析没踩到就放行了。修法不是放弃独立性，而是区分"不得被上一轮的分析锚定"与"必须核验上一轮阻断了什么"——后者是可机械核验的事实。
- **子代理是后台异步的。** 控制会话收到"子代理已在后台运行"不等于任务完成。LS-014 因此提前返回了一份与真实提交矛盾的 BLOCKED 快照。
