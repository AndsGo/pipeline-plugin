# 参与开发

本仓库是 Markdown 与一份 JSON Schema，没有构建步骤。但 skill 集是"需要治理的活物"——改一个词可能让下游阶段的行为漂移，所以纪律比一般代码仓库更严。

## 改之前

1. 读 [`CLAUDE.md`](CLAUDE.md)：登记表、变更纪律、词汇约定、每个 skill 的健康判据。
2. 读 [`skills/_shared/handoff-spec.md`](skills/_shared/handoff-spec.md)：所有 skill 共用的契约。
3. 看 [`docs/writing-a-skill.md`](docs/writing-a-skill.md)。

## 三类改动，三种流程

| 改动 | 需要 | 版本 |
|---|---|---|
| **skill 内容修订**（判据补强、措辞、加一条纪律） | 跑评估集回归；如果源于真实事故，优先把事故做成新的评估场景 | 补丁位 +1 |
| **契约字段变更**（schema 加字段、frontmatter 加键） | 同步改 `schemas/stage-result.schema.json` 与 `handoff-spec.md`（含顶部变更史）；编排器仓库 `src/types.ts` 同时出 PR | 次版本 +1，两个仓库同步 |
| **skill 增删改名** | 更新 `CLAUDE.md` 登记表；编排器 `runner.ts` 以 skill 名拼指令、`config.ts` 有对应白名单——同时改 | 次版本 +1 |

废弃即删除，不留尸体目录；删除的提交信息注明替代者。

## 写作纪律（PR 会按这个审）

- **每个动作步骤带完成判据**。"充分评审"不合格，"每个 hunk 被两轴各查一次"合格。锐边界 + 强工作量。
- **说目标行为，不说禁行为**——除了独立性这类必须以"禁止读取"表达的边界。
- **一词一义**。用词优先取自 `CLAUDE.md` 的词汇约定：驳回（人在卡点否决）≠ 打回（BLOCK 触发的修复回环）；工件 ≠ 产物；AC 清单 ≠ 检查单；findings 用英文原词。
- **引用事故**。一条纪律为什么存在，写它防的是哪张工单（现有 skill 里满是 `LS-012 实测`）。
- **外部依赖点名**。依赖 superpowers 的哪个 skill、改了它哪几条，写在 skill 开头。

## 评估集

```bash
cd ../pipeline-orchestrator
npx tsx scripts/skill-eval.ts --all
```

改 skill、换模型、升级 superpowers 后**必须**回归。新增场景写法见 [`evals/README.md`](evals/README.md)。

## 提交信息

```
feat(acceptance): 人工 AC 追问设硬上限——两轮无新输入即收敛为「无法验证 + 上线后补验」，不再无限循环（0.14.0）
```

前缀 `feat / fix / docs / chore`，括号里是 skill 名或 `handoff` / `schema` / `plugin`，标题末尾带版本号。

## 报告问题

skill 行为不对时，issue 里带：工单的交接文档（`docs/pipeline/<ticket>/` 相关文件）、结构化返回、用的引擎与模型、`handoff-spec` 版本。能复现的话，把它做成一个评估场景直接开 PR 更好。
