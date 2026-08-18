---
ticket: T-1
stage: clarify
status: DONE
produced: 2026-08-18T10:00:00+08:00
inputs:
  - 00-intake.md
next_stage_reads:
  - 10-prd.md
---

## 本阶段结论

给 calc 模块补齐加法与减法两个函数，各一条 AC。

## 背景与目标

内部计算模块需要基础四则里的加减两项能力。

## 范围

**In：** add 函数、sub 函数
**Out：** 乘除、任何 UI

## 需求描述

- R-1：提供 add(a,b) 返回 a+b
- R-2：提供 sub(a,b) 返回 a-b

## 验收标准（AC 清单）

### AC-1: add 函数正确（验证 R-1）
**Given** calc 模块已加载
**When** 调用 add(1,2)
**Then** 返回 3
**业务验收描述：** 加法算得对
**验证方式：** 自动 — `node -e "const {add}=require('./src/calc.js'); process.exit(add(1,2)===3?0:1)"`

### AC-2: sub 函数正确（验证 R-2）
**Given** calc 模块已加载
**When** 调用 sub(5,2)
**Then** 返回 3
**业务验收描述：** 减法算得对
**验证方式：** 自动 — `node -e "const {sub}=require('./src/calc.js'); process.exit(sub(5,2)===3?0:1)"`

## 非功能要求
无

## 关键决策及理由
无

## 事实 vs 假设
**已验证事实：**
- src/calc.js 存在且为 CommonJS 模块

**未验证假设：**
无假设

## 失败过的尝试
无

## 阻塞与开放问题
无

## 给下一阶段的指令
读本文件全文，计划的每个任务必须声明覆盖哪些 AC。
