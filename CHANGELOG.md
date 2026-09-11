# 变更日志

`plugin.json` 的主次版本跟随 `skills/_shared/handoff-spec.md` 的契约版本，补丁位留给不动契约的 skill 内容修订。括号里的「契约」是 handoff-spec 版本。每条尽量注明它来自哪张真实工单。

## [Unreleased]

### 文档
- 开源化：新增 README、LICENSE（MIT，含 diagnosing-bugs 第三方声明）、CONTRIBUTING、CHANGELOG，`docs/writing-a-skill.md`、`docs/contract.md`、`evals/README.md`。

## [0.16.0] — 2026-09-09（契约 0.12.0）
- **handoff**：新增 `stale_hints`——阶段核实到 `05-knowledge-hints.md` 条目与现状矛盾时回报标题（逐字），编排器据此标「待复核」停注入、闭环发卡定夺。

## [0.15.0] — 2026-09-09
- **acceptance**：识别「点了按钮没打字」的非回答，当场点破而非采信。

## [0.14.0] — 2026-09-09
- **acceptance**：人工 AC 追问设硬上限——两轮无新输入即收敛为「无法验证 + 上线后补验」，不再无限循环。

## [0.13.0] — 2026-09-05
- **acceptance**：浏览器实测一步——项目开了 `e2e: playwright` 时，页面类人工 AC 先用 Playwright MCP 实测，判不了的再留给人。

## [0.12.1] — 2026-09-04
- **implement**：子代理未收工不得返回结构化结果——LS-014 控制会话在子代理仍在后台跑时提前返回 BLOCKED，与子代理的真实提交矛盾。返回前用 `git log` / `git status` 核对每个子代理都已结束且已提交。

## [0.12.0] — 2026-09-03（契约 0.12.0）
- **skills**：各阶段先读 `07-project-profile.md`——目标仓库 `PIPELINE.md` 的项目流程约定节选，项目事实优先于通用流程。六个阶段 skill 同步。

## [0.11.1] — 2026-09-01
- **prototype**：新增 `pipeline-prototype`——定稿 PRD → 一页业务人员看得懂的结果预览（HTML），PRD 确认卡带预览链接。

## [0.11.0] — 2026-08-22 ～ 08-25（契约 0.11.0）
- **review**：固化 LS-012 三条评审教训——不得被上一轮分析锚定但必须核验上一轮阻断了什么；横切改动全入口核查；既往 BLOCK 未经修复轮不得放行。
- **system-map**：新增 `pipeline-system-map`——目标仓库能力地图，给 agent 和业务人员共用的系统入口；compound 负责增量维护。
- **plan**：重入判据——已有完整 `20-plan.md` 时校验后补返回，不重写。

## [0.10.3] — 2026-08-24
- **implement**：子代理模型分层由建议升为硬约束；版本位回到补丁位。

## [0.10.1] — 2026-08-19
- **skills**：钉版本引入 `diagnosing-bugs`（mattpocock/skills，MIT），修复轮可自主够到的诊断纪律层；`PROVENANCE.md` 记上游 commit 与本地修改。

## [0.10.0] — 2026-08-18（契约 0.10.0）
- **contract**：review 的 `axes` 双轴清点（Spec / Quality，永不合并）成为完成态必填。
- **evals**：评估集与 `expect.json` 断言格式，首批两个场景（`review-blocks-missed-ac`、`acceptance-blocks-failing-auto-ac`）。
- **CLAUDE.md**：词汇约定（驳回 ≠ 打回、工件 ≠ 产物、AC 清单 ≠ 检查单）。

## [0.9.1] — 2026-08-18
- **skills**：A 批——每个动作步骤带完成判据；七个阶段 skill 关闭模型自主触发（`disable-model-invocation: true`）；新增维护元规范 `CLAUDE.md`。

## [0.9.0] — 2026-08-18（契约 0.9.0）
- **spec**：规则 4「流程主权」——本契约与阶段 skill 优先于仓库 CLAUDE.md 的工作流规则与其他插件的元技能；「问人」的唯一合规形式是 `NEEDS_CONTEXT`。

## [0.8.1] — 2026-08-18
- **skills**：采纳 LS-009 两条流程改进建议。

## [0.8.0] — 2026-08-17（契约 0.8.0）
- **clarify**：PM 主动访谈升级 + 术语采集（`93-terms.json`）+ 项目简报。

## [0.7.0] — 2026-08-15（契约 0.7.0）
- 插件入库：clarify / plan / implement / review / acceptance / compound 六个阶段 skill、`handoff-spec.md`、`stage-result.schema.json`。

[Unreleased]: https://github.com/AndsGo/pipeline-plugin/compare/v0.16.0...HEAD
[0.16.0]: https://github.com/AndsGo/pipeline-plugin/releases/tag/v0.16.0
