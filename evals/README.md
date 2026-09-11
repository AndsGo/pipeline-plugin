# 评估集

改 skill、换模型、升级 superpowers 后**必须**回归。运行器在编排器仓库：

```bash
cd ../pipeline-orchestrator
npx tsx scripts/skill-eval.ts --all                      # 全量，sonnet 降费执行，约 $1
npx tsx scripts/skill-eval.ts review-blocks-missed-ac    # 单个场景
```

## 一个场景的结构

```
evals/<场景名>/
├── expect.json      断言（必有）
├── shared/          复制进临时工作区的文件：工件（docs/pipeline/T-1/…）与源码
├── base/            可选：构造 diff 的基线版本（review 类场景）
└── head/            可选：构造 diff 的改动后版本
```

运行器在临时 git 仓库里铺好 `shared/`，若有 `base/` `head/` 则先提交 base 再提交 head 得到一个真实 diff，然后调起对应 skill 并把结构化返回与 `expect.json` 比对。

## expect.json

```json
{
  "skill": "review",
  "why": "两条 AC 只实现一条——评审必须 BLOCK 且 Spec 轴清点出未实现项（不得因『总体不错』软化）",
  "extraArgs": "base={BASE_SHA}",
  "assert": {
    "status": ["DONE", "DONE_WITH_CONCERNS"],
    "verdict": "BLOCK",
    "axes_spec_failed_min": 1
  }
}
```

| 字段 | 说明 |
|---|---|
| `skill` | 阶段名（不带 `pipeline-` 前缀） |
| `why` | 这个场景守的是哪条纪律、来自哪个真实事故。**必写**——半年后没人记得为什么要有它 |
| `extraArgs` | 传给 skill 的额外参数；`{BASE_SHA}` 由运行器替换为 base 提交 |
| `assert.status` | 允许的 status 列表 |
| `assert.verdict` | 期望的 verdict |
| `assert.axes_spec_failed_min` | Spec 轴 `failed` 的下限 |

## 现有场景

| 场景 | 守什么 |
|---|---|
| `review-blocks-missed-ac` | 两条 AC 只实现一条 → 必须 BLOCK 且 Spec 轴清出未实现项，不得因"总体不错"软化 |
| `acceptance-blocks-failing-auto-ac` | 自动 AC 跑失败 → 必须 BLOCK，不得靠实现报告自述放行（LS-008 的 AC-8） |

## 新增场景

优先取自真实工单的事故：把当时的工件裁成最小复现放进 `shared/`，`why` 写清事故编号与现象。一个场景只守一条纪律。
