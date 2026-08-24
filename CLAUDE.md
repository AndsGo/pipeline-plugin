# pipeline-plugin 维护规范

本插件是飞书流水线的阶段技能集。**skill 集是需要治理的活物**——本文件是登记表、变更纪律与健康判据的单一真源。

## Skill 登记表

| skill | 一句话职责 | 调用方 |
|---|---|---|
| pipeline-clarify | 原始需求 → PRD + 业务语言 AC 清单 + 新术语提议（93-terms.json） | 仅编排器 |
| pipeline-plan | PRD → 零上下文可执行的任务计划（垂直切片、依赖有序） | 仅编排器 |
| pipeline-implement | 计划 → 代码 + 测试 + MR（薄编排，纪律委托 superpowers SDD；台账位置/无人可问/子代理模型分层三条本地覆盖） | 仅编排器 |
| pipeline-review | 独立双轴评审：Spec 符合性 + 质量 findings（带失败场景） | 仅编排器 |
| pipeline-acceptance | AC 清单 → 自动项亲跑取证 + 人工检查单（业务语言） | 仅编排器 |
| pipeline-compound | 全套工件 → 交付文档 / 知识条目 / 建议（92/93/95/96）+ PROJECT-BRIEF 维护 | 仅编排器 |
| diagnosing-bugs | 疑难 bug 六阶段门控诊断（修复轮引用；**钉版本外来件**，见其 PROVENANCE.md，升级须手动 diff 上游 + 跑 skill-eval） | 模型可调 |
| _shared/handoff-spec.md | 全体 skill 的交接契约（版本单一真源之一） | 被引用 |
| schemas/stage-result.schema.json | 结构化返回契约 | 编排器 --json-schema |

六个 pipeline-* 阶段 skill 均 `disable-model-invocation: true`——只由编排器显式调起，交互式会话里被模型自主触发属于事故。**例外：diagnosing-bugs 保持模型可调**——它是纪律层，修复轮的会话需要自主够到它。

## 变更纪律

1. **契约字段变更**：必须同步改 `schemas/stage-result.schema.json` 并升 handoff-spec 版本号（spec 内有变更史）。`plugin.json` 的主次版本跟随 spec，**补丁位**留给不动契约的 skill 内容修订（如 0.9.1 = spec 0.9.0 + 判据补强）。
2. **skill 增删改名**：同步更新本登记表与编排器侧的调用（`runner.ts` 以 skill 名拼指令）。
3. **废弃即删除**：不留尸体目录；删除的提交信息注明替代者。
4. **写作纪律**：每个动作步骤必须带完成判据（锐边界 + 强工作量——"充分评审"不合格，"每个 hunk 被两轴各查一次"合格）；说目标行为而不是禁行为；同一含义只允许一个措辞（用词优先取自项目术语表与 handoff-spec 既有词汇）。
5. **外部依赖要点名**：implement 依赖 superpowers:subagent-driven-development、plan 改编自 superpowers:writing-plans——上游升级属于行为变更，升级后须过一遍试跑。

## 词汇约定（leading words——一词一义，写作时按此选词）

- **驳回** = 人在卡点上的否决（prd-confirm/plan-approval/deploy-approval/建议卡）；**打回** = review/acceptance 的 `BLOCK` 触发的修复回环。二者不可混用。
- **工件** = `docs/pipeline/<ticket>/` 下随分支提交的文件本体；**产物** = 一个阶段产出的统称（工件 + 结构化返回）。
- **AC 清单** = PRD 里的验收标准表；**检查单** = acceptance 发给人的人工验收问卷。
- **findings** 固定用英文原词（评审发现项），不写「问题/缺陷/发现」。

## 评估集（evals/）

`evals/<场景名>/` = `expect.json`（断言）+ `shared/`（工作区文件）+ 可选 `base/`、`head/`（构造 diff）。跑法：编排器仓库 `npx tsx scripts/skill-eval.ts --all`（sonnet 降费执行，约 $1/全量）。**改 skill 或换模型后必须回归**；新增回归场景优先取自真实工单的事故（如 LS-008 的 AC-8）。

## 每个 skill 的健康判据（it's working if）

- **clarify**：PRD 零实现细节；澄清常在一轮内收敛；AC 全带「业务验收描述」；术语提议不含通用词。
- **plan**：implement 阶段零 NEEDS_CONTEXT（信息在计划里就是全的）；SDD 子代理单任务不超上下文。
- **implement**：ledger 可断点恢复；MR 建得出（无 glab 走 push options）；对 finding 零静默丢弃。
- **review**：findings 全带 file:line + 失败场景；写不出失败场景的自动降级 Minor；横切改动有全入口核查记录。
- **acceptance**：「无法验证」诚实保留，从不冒充通过；「不通过」必附现象。
- **compound**：同一条建议不会第二次被提出（再次出现 = 上游采纳通道断了，先修通道）；知识条目 scope 填充率 100%。

判据恶化即回归信号——对照编排器 /dashboard 的三指标（评审一次通过率 / 修复轮 / 验收返工）定位是哪个 skill 松了。
