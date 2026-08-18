---
ticket: T-1
stage: plan
status: DONE
produced: 2026-08-18T10:10:00+08:00
inputs:
  - 10-prd.md
next_stage_reads:
  - 20-plan.md
---

## 审批摘要
在 src/calc.js 实现 add 与 sub 两个函数，覆盖 AC-1、AC-2。

## 本阶段结论
单文件两函数，一个任务完成。

## Global Constraints

- 分支：feat/T-1-calc
- 只允许修改 src/calc.js
- Conventional Commits
- 每个函数必须被 AC 验证命令覆盖

### Task 1: calc 模块（覆盖：AC-1, AC-2）
实现 add 与 sub 并导出。

## 关键决策及理由
无

## 事实 vs 假设
**已验证事实：** 无
**未验证假设：** 无假设

## 失败过的尝试
无

## 阻塞与开放问题
无

## 给下一阶段的指令
按 Task 1 实现后交评审。
