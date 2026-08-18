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
calc 模块需要一个乘法函数，一条自动 AC。

## 背景与目标
内部计算模块补乘法。

## 范围
**In：** mul 函数
**Out：** 其他运算

## 需求描述
- R-1：提供 mul(a,b) 返回 a*b

## 验收标准（AC 清单）

### AC-1: mul 函数正确（验证 R-1）
**Given** calc 模块已加载
**When** 调用 mul(2,3)
**Then** 返回 6
**业务验收描述：** 乘法算得对
**验证方式：** 自动 — `node -e "const {mul}=require('./src/calc.js'); process.exit(mul(2,3)===6?0:1)"`

## 非功能要求
无

## 关键决策及理由
无

## 事实 vs 假设
**已验证事实：**
- src/calc.js 存在

**未验证假设：**
无假设

## 失败过的尝试
无

## 阻塞与开放问题
无

## 给下一阶段的指令
验收按 AC 清单执行。
