---
name: pipeline-plan
description: 流水线阶段③——制定实现计划。当工单的 PRD（10-prd.md）已经业务确认、需要产出可被零上下文会话逐任务执行的实现计划时使用。调用参数为工单 ID（如 /pipeline-plan TICKET-123）。
---

# Pipeline Plan：制定实现计划

> 改编自 superpowers:writing-plans（v6.2.0），适配流水线交接契约与内网 GitLab 环境。

## 概述

为**零上下文、不了解本代码库**的执行者编写实现计划：每个任务要动哪些文件、完整的代码、怎么测、怎么提交。全部拆成 2-5 分钟粒度的步骤。DRY、YAGNI、TDD、频繁提交。

**开始前必读**：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/handoff-spec.md`。

**参数**：工单 ID。**输入**：`docs/pipeline/<ticket-id>/10-prd.md`（frontmatter 必须是 `stage: clarify`、`status: DONE` 或 `DONE_WITH_CONCERNS`；不满足 → `BLOCKED`，说明输入未就绪）。**输出**：`docs/pipeline/<ticket-id>/20-plan.md`。

**工具边界**：只读代码库 + 写计划文件。本阶段不改任何业务代码、不建分支——那是实现阶段的事。

**历史知识**：工单目录存在 `05-knowledge-hints.md` 时先读它。与本次改动相关的"踩坑"条目必须转化为计划里的**具体动作**（写进对应任务的步骤或 Global Constraints），不能只是知道；明确不适用的在"关键决策及理由"里说明。

**术语**：工单目录存在 `06-glossary.md` 时先读它。计划中的业务名词与 PRD、术语表保持一致，不得引入新叫法。

**人工反馈**：调用参数含 `feedback=<path>`（通常是审批人驳回了上一版计划）时，按 handoff-spec §8 先读该文件；新计划必须逐条回应驳回意见，并在"关键决策及理由"里写明改了什么。被驳回的旧方案进"失败过的尝试"，不得原样重提。

**异常出口**：
- PRD 有计划无法自行消解的歧义（缺关键信息）→ `NEEDS_CONTEXT` + open_questions（同样要带推荐答案）。
- PRD 与代码现状矛盾（如 PRD 假设的模块不存在、AC 与既有行为冲突）→ `BLOCKED`，写清矛盾双方与建议解法。**不要替业务方做决定后继续写计划。**

## 范围检查

如果 PRD 覆盖多个相互独立的子系统，建议拆成多个工单（每个工单独立产出可工作、可测试的软件），在"阻塞与开放问题"里提出并 `DONE_WITH_CONCERNS`；若耦合到无法拆分，照常写一份计划。

## 文件结构先行

定义任务前，先画出将创建/修改的文件清单及各自职责——分解决策在这里锁定：

- 单元边界清晰、接口明确，每个文件一个职责。
- 偏向小而聚焦的文件。
- 一起变化的代码放一起，按职责拆分，不按技术分层拆分。
- 存量代码库跟随既有模式；不要顺手重构没坏的东西。

## 任务尺寸

任务是**携带独立测试循环、值得一次独立评审**的最小单元：配置、脚手架、文档步骤折叠进需要它们的任务；只在"评审人可以否决这个任务却通过邻居"的地方切分。每个任务以可独立验证的交付物收尾。

## 步骤顺序红线

任务步骤中出现「先执行不可逆副作用（吊销/撤销凭据、删除数据、自增版本计数、发外部通知），后对输入做合法性校验」的顺序时，必须显式检查「校验失败但副作用已生效」的组合：把校验前置，或写明为何该顺序安全。（LS-009 曾把有效期校验排在轮换计数器自增之后——超限输入会让旧 Key 已作废而新 Key 未签发；此类缺陷在计划阶段拦截成本最低。）

**步骤粒度（每步一个动作，2-5 分钟）**：写失败测试 → 跑确认失败 → 最小实现 → 跑确认通过 → 提交，各是一步。

## 计划文档结构

`20-plan.md` 的 frontmatter 按 handoff-spec §2.1（`stage: plan`），正文如下：

````markdown
## 审批摘要
[给审批人的决策材料，≤15 行：改动范围（模块/文件数）、任务数、
主要风险与缓解、回滚方式（如 revert MR）、预估不确定点。
编排器把这段贴到飞书审批单上，审批人可能只读这一段。]

## 本阶段结论
[一句目标 + 架构方式 2-3 句 + 任务数与顺序概览]

# [功能名] 实现计划

> **执行者须知：** 用 superpowers:subagent-driven-development 逐任务执行本计划。
> 步骤用 checkbox（`- [ ]`）跟踪。进度台账写入本工单目录的 ledger.md 并随任务提交。

**目标：** [一句话]
**架构：** [2-3 句]
**技术栈：** [关键技术/库]
**覆盖验收标准：** [AC-1, AC-2, ...（必须与 10-prd.md 的 AC 清单一一对应，缺一条即为计划缺陷）]

## Global Constraints

[从 PRD 逐字抄出的全局要求：版本下限、命名与文案规则、平台要求，每条一行。
每个任务隐式继承本节。以下为流水线固定条目，每份计划都包含：]
- Git：分支名 `feat/<ticket-id>-<slug>`；提交信息遵循 Conventional Commits；禁止直接提交到 main
- GitLab 操作一律用 glab CLI（执行机已配 GITLAB_HOST / GITLAB_TOKEN），完成后创建 MR，不直接 merge
- 进度台账：docs/pipeline/<ticket-id>/ledger.md，随任务完成一起 commit

---

### Task N: [组件名]（覆盖：AC-3, AC-5）

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [使用前序任务的什么——精确签名]
- Produces: [后续任务依赖的什么——精确函数名、参数与返回类型。
  执行者只看到自己的任务，这个块是它得知邻居任务所用名字和类型的唯一途径]

- [ ] **Step 1: 写失败测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: 运行确认失败**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL，报 "function not defined"

- [ ] **Step 3: 最小实现**

```python
def function(input):
    return expected
```

- [ ] **Step 4: 运行确认通过**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```

[...全部任务...]

## 关键决策及理由
## 事实 vs 假设
## 失败过的尝试
## 阻塞与开放问题
## 给下一阶段的指令
[通常是：按"执行者须知"用 SDD 执行；ledger 路径；MR 创建方式；哪些 AC 是自动验证、对应哪些任务的测试]
````

## No Placeholders 红线

每一步必须包含执行者需要的实际内容。以下写法是**计划缺陷**，一律禁止：

- "TBD"、"TODO"、"稍后实现"、"补充细节"
- "添加适当的错误处理" / "补充校验" / "处理边界情况"
- "为上面的写测试"（不给出实际测试代码）
- "类似 Task N"（把代码原地重复——执行者可能乱序阅读）
- 只说做什么不给怎么做（代码步骤必须有代码块）
- 引用任何任务都没定义过的类型、函数、方法

## Self-Review（写完后自查，不派子代理）

1. **AC 覆盖**：逐条过 10-prd.md 的 AC 清单——每条能指到至少一个声明覆盖它的任务吗？头部"覆盖验收标准"列表与逐任务标注一致吗？列出缺口并补任务。
2. **占位符扫描**：按上节红线逐条扫计划全文，发现即修。
3. **类型一致性**：后面任务用到的类型、方法签名、属性名与前面任务定义的一致吗？（Task 3 叫 `clearLayers()`、Task 7 叫 `clearFullLayers()` 就是 bug。）
4. **Global Constraints 逐字性**：约束是从 PRD 逐字抄的，还是自己转述的？转述的改回逐字。

发现问题原地修复，修完即过，不用重复自查。

## 结束

按 handoff-spec §6 自查后输出结构化返回。`summary_for_card` 面向审批人：改什么、多少任务、风险、回滚方式（可从"审批摘要"压缩）。

本阶段 DONE 后由编排器发起飞书 Approval 正式审批，**审批通过前实现阶段不会启动**——所以审批摘要里的风险陈述要诚实，那是人类拦住问题的最后一道低成本关口。
