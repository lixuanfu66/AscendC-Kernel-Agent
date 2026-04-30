# CANN Agent Baseline

本目录记录基线代码构建方案。

基线选择：

```text
主干：D:\code\CannBot
上游 skill：external/cann-skills
本项目新增：memory、strategy pool、verifier summaries、dataset export
```

详细方案见：

- `wiki/baseline_integration_plan_ascendc_kernel_agent_with_cann_skills.md`
- `baseline/BASELINE_v0.md`

## Baseline v0

当前基线定义见：

- `baseline/BASELINE_v0.md`

## 第一版 skill 状态

经确认，以下 10 个核心 skill 已经存在于根目录 `.claude/skills/` 中，不需要重复引入：

- `ascendc-docs-search`
- `ascendc-api-best-practices`
- `ascendc-tiling-design`
- `ascendc-npu-arch`
- `ascendc-env-check`
- `ops-precision-standard`
- `ascendc-precision-debug`
- `ascendc-runtime-debug`
- `ops-profiling`
- `ascendc-code-review`

保留主干已有：

- `ascendc-msopgen-workflow`

当前重点不是复制 skill，而是做：

1. 与 `external/cann-skills/ops/` 的上游差异审计。
2. 判断哪些上游更新需要同步。
3. 保留 `AscendC-Kernel-Agent` 的本地增强 skill，例如 `ascendc-kb-docs` 和 `ascendc-msopgen-workflow`。
4. 只补充主干缺失且有价值的可选 skill，例如 `torch-ascendc-op-extension`、`ascendc-docs-gen`、`ascendc-performance-best-practices`、`ascendc-regbase-best-practice`。

## 第一版目标

1. 跑通或结构化失败 `scoring/score.sh`。
2. 每个 verifier 阶段输出 JSON summary。
3. 每个 attempt 绑定 strategy id。
4. 每轮 score 触发 memory update 和 trajectory export。
