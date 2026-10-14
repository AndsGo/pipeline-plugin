# pipeline-plugin

> **English summary** — The stage skills and the JSON contract for [pipeline-orchestrator](https://github.com/AndsGo/pipeline-orchestrator). Each skill is the job description plus the non-negotiable discipline for one SDLC stage (clarify, plan, implement, review, acceptance, compound, prototype, system-map), written as a Claude Code plugin skill. Skills hand off through git-committed artifacts with a fixed shape (frontmatter + six sections) and return a four-state structured result validated against `schemas/stage-result.schema.json`. The same skills run under Claude Code natively or under Codex CLI through a bridge prompt. Chinese-first.

飞书 + Claude Code / Codex 开发流水线的**阶段 skill 集与交接契约**。编排器（[pipeline-orchestrator](https://github.com/AndsGo/pipeline-orchestrator)）负责"下一步跑什么"，本仓库负责"每一步怎么干、干到什么程度算完"。

一个 skill 不是提示词模板，而是**一个角色的岗位说明书 + 不许越界的纪律**。评审 skill 只有 68 行，其中最重要的一节叫「独立性纪律（本 skill 的存在理由）」——它划定的是不许看什么，而不是该怎么评。

## 组成

```
skills/
├── _shared/handoff-spec.md      交接契约：目录约定、机器头 + 六节人类体、四态语义、写作纪律、自查清单
├── pipeline-clarify/            ② 原始需求 → PRD + 业务语言验收标准 + 新术语提议
├── pipeline-prototype/          ②′ 定稿 PRD → 一页业务人员看得懂的结果预览（HTML）
├── pipeline-plan/               ③ PRD → 零上下文可执行的任务计划
├── pipeline-implement/          ④ 计划 → 代码 + 测试 + MR（子代理驱动，台账可断点恢复）
├── pipeline-review/             ⑤ 独立双轴评审：Spec 符合性 + Quality findings
├── pipeline-acceptance/         ⑦ 验收标准 → 自动项亲跑取证 + 人工检查单
├── pipeline-compound/           ⑧ 全套工件 → 交付文档 / 知识 / 术语 / 建议 + 项目简报与能力地图维护
├── pipeline-system-map/         目标仓库能力地图的全量重建与增量
└── diagnosing-bugs/             疑难 bug 诊断纪律（外来件，钉版本，见 PROVENANCE.md）
schemas/stage-result.schema.json 结构化返回契约
evals/                           评估集：改 skill 或换模型后必须回归
CLAUDE.md                        维护规范：登记表、变更纪律、词汇约定、健康判据
```

阶段⑥（CI）由编排器原生执行，不花模型，没有 skill。

## 契约（一分钟版）

每个 skill 在 headless 调用下以 `--json-schema schemas/stage-result.schema.json` 约束最终输出：

| 字段 | 说明 |
|---|---|
| `status` | `DONE` / `DONE_WITH_CONCERNS` / `NEEDS_CONTEXT`（缺信息，带 ≥1 个问题，每题必有推荐答案）/ `BLOCKED`（有障碍，带理由） |
| `verdict` | review / acceptance 完成态必填：`PASS` / `PASS_WITH_SUGGESTIONS` / `BLOCK`。与 status **正交**——"评审顺利完成、结论是阻断"是 `DONE` + `BLOCK` |
| `axes` | review 必填：`{spec:{total,failed,worst}, quality:{critical,important,minor,worst}}`，两轴永不合并 |
| `handoff_path` | 交接文档路径；**文件为准**，结构化返回只是路由信号 |
| `summary_for_card` | ≤500 字，编排器原样渲染进飞书卡片，写给业务方看 |
| `stale_hints` | 可选：核实到与现状矛盾的历史知识条目标题 |

四条写作硬规则：无占位符（"TBD"、"同上"、"如前所述"都算缺陷）；**事实 vs 假设强制分区**；零上下文假设（不了解代码库的人拿本文档就能开工）；**流程主权**（本契约与阶段 skill 优先于仓库 CLAUDE.md 的工作流规则与其他插件的元技能；"问人"的唯一合规形式是 `NEEDS_CONTEXT`）。

全文见 [`skills/_shared/handoff-spec.md`](skills/_shared/handoff-spec.md)，当前版本 0.12.0，变更史在文件顶部。

## 安装与使用

**作为编排器的一部分**（正常用法）：与 pipeline-orchestrator 平级 clone，编排器默认在 `../pipeline-plugin` 找它，或设 `PIPELINE_PLUGIN_DIR`。编排器用 `claude -p '/pipeline-<stage> <ticket>' --plugin-dir <本目录>` 调起。

**单独调起一个阶段**（调试）：

```bash
cd <目标仓库>
claude -p '/pipeline-review LS-014 base=<sha>' \
  --plugin-dir /path/to/pipeline-plugin \
  --allowedTools 'Read,Grep,Glob,Write,Bash' \
  --output-format json \
  --json-schema "$(node -e "const s=require('/path/to/pipeline-plugin/schemas/stage-result.schema.json');delete s.allOf;console.log(JSON.stringify(s))")"
```

注意 `--json-schema` 传的是剥掉顶层 `allOf` 的版本（接口限制），完整校验由编排器回程做。

**在 Codex 下运行**：编排器的 codex 引擎会把 `/pipeline-<stage>` 桥接成"先完整读 `skills/<stage>/SKILL.md` 再照做"，并把 schema 转成严格形态。skill 本身不需要为 Codex 改写。

六个阶段 skill 与 `pipeline-system-map` 都是 `disable-model-invocation: true`——只由编排器显式调起，交互式会话里被模型自主触发属于事故。两个例外：`pipeline-prototype` 可在交互会话里手动调起给业务方出预览；`diagnosing-bugs` 是修复轮要自主够到的纪律层。

## 依赖

- **superpowers**（Claude Code 插件）：`pipeline-implement` 以 `superpowers:subagent-driven-development` 为执行引擎（有四条本地覆盖规则：台账位置、无人可问、子代理模型分层为硬约束、产物出口判据），`pipeline-plan` 改编自 `superpowers:writing-plans`。上游升级属于行为变更，须跑评估集。
- **不要**同时启用 engineering-workflow 一类会向每个会话注入自身流程规则的元框架——两层流程控制会静默打架（曾把它的流程要求写进 PRD）。编排器的 `config/pipeline-settings.json` 已禁用。

## 评估集

```bash
# 在 pipeline-orchestrator 仓库里
npx tsx scripts/skill-eval.ts --all             # 全量，sonnet 降费，约 $1
npx tsx scripts/skill-eval.ts review-blocks-missed-ac
```

`evals/<场景>/` = `expect.json`（断言）+ `shared/`（工作区文件）+ 可选 `base/` `head/`（构造 diff）。**改 skill 或换模型后必须回归**；新场景优先取自真实工单的事故。见 [`evals/README.md`](evals/README.md)。

## 版本

`plugin.json` 的主次版本跟随 `handoff-spec.md` 的契约版本，补丁位留给不动契约的 skill 内容修订（0.12.1 = 契约 0.12.0 + 判据补强）。当前候选版 **0.16.3-rc.1**（契约 0.12.0），配套编排器 **0.8.0-rc.1**。历史见 [CHANGELOG.md](CHANGELOG.md)。

## 文档

- [写一个 skill](docs/writing-a-skill.md) — 结构、纪律、自查、评估
- [契约](docs/contract.md) — 两个形态的 schema、版本升级流程
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [CLAUDE.md](CLAUDE.md) — 维护规范（登记表、变更纪律、每个 skill 的健康判据）

## 许可

[MIT](LICENSE)。`skills/diagnosing-bugs/` 来自 [mattpocock/skills](https://github.com/mattpocock/skills)（MIT，Matt Pocock），钉定 commit 与本地修改见其 `PROVENANCE.md`。

## 多 CLI 编排接入

2026-10-13 编排器按用户要求将默认底座改为 Codex；Claude Code 与 OpenCode 可按全局、项目与阶段显式选择。编排器将本插件同一套阶段规则交付给各适配器，已有工单继续使用冻结配置。共享规则读取 `AGENTS.md` 与 `docs/pipeline/PROJECT-KNOWLEDGE.md`；旧 `CLAUDE.md` 保留为兼容输入。路径变量使用 `${PIPELINE_PLUGIN_ROOT}`，Claude 旧变量由桥接兼容。

适配器必须真实提供规则要求的权限、预算、轮数、浏览器及角色子代理能力；缺失时报告 BLOCKED，不得模拟或自动换 CLI。当前 Codex/OpenCode 的必需约束仍有未验收能力，注册适配器不等于已经可用于生产。执行配置及验收限制见 sibling orchestrator 的 `docs/agent-execution.md`。
