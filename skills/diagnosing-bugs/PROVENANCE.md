# 来源与钉版本记录

- **上游**：https://github.com/mattpocock/skills — `skills/engineering/diagnosing-bugs/`
- **钉定 commit**：`9c9f36ccd3995266cd675468af71639c8dde1ec5`（2026-08-19 拷入）
- **许可**：MIT（原文见同目录 `LICENSE.upstream`，版权 Matt Pocock）
- **引入理由**：修复轮需要「先建反馈回路再动手」的门控纪律；该 skill 环境中立、模型可调，是少数可直接复用的外部 skill。
- **本地修改（仅 1 处）**：SKILL.md 第 10 行的 `CONTEXT.md`/ADR 指引改为指向本流水线的 `06-glossary.md` 与 `docs/pipeline/PROJECT-BRIEF.md`。其余逐字节与上游一致。
- **升级纪律**：不追踪上游（其仓库有删改历史）。如需升级：`git -C D:\git\mattpocock\skills pull` 后手动 diff 上游同路径，评估变更、重放本地修改、更新本文件的 commit 记录，**并跑 skill-eval 回归**。
