# CANN 算子编程 Agent Baseline v0

## 1. 基线定位

Baseline v0 的目标不是重新造一个 Agent，而是在 `AscendC-Kernel-Agent` 现有能力上建立一个可演进研究基线：

```text
AscendC-Kernel-Agent 现有主干
  + 已内置 CANN 核心 skills
  + scoring/verifier 闭环
  + lineage 工作区
  + 后续新增 memory / strategy / dataset export
```

Baseline v0 先保证：

- 可以描述一个算子任务。
- 可以生成或修改 Ascend C custom operator 工程。
- 可以通过统一 scoring pipeline 做编译、部署、正确性和性能验证。
- 可以保留每轮尝试的日志、分数、失败原因和版本谱系。
- 后续可以接入经验库、value-aware memory 和训练语料导出。

## 2. 主干选择

主干：

```text
D:\code\CannBot
```

理由：

- 已实现 AVO 式框架：`Vary(P_t)=Agent(P_t,K,f)`。
- 已有 Agent Team：Architect、Developer、Reviewer、Tester、Supervisor、Reporter。
- 已有 custom operator 工程路线。
- 已有 `scoring/score.sh` 分层评分管线。
- 已有 `evolution/` 状态、分数、日志目录。
- 已有 `workspace/runs/{op}/best` 与 `attempts/step_N` 工作区模型。

上游参考：

```text
external/cann-skills
```

用途：

- 作为 CANN skills 的上游版本来源。
- 用于差异审计和按需同步。
- 不作为默认全量上下文导入。

## 3. 已内置核心 Skills

经确认，`AscendC-Kernel-Agent/.claude/skills/` 已包含第一版需要的 10 个核心 skill：

| Skill | 作用 | 接入阶段 |
| --- | --- | --- |
| `ascendc-docs-search` | API、样例、兼容性检索 | Architect / Developer / Reviewer |
| `ascendc-api-best-practices` | API 规则、DataCopy、Buffer、Pipeline | Developer / Reviewer / Long-term memory |
| `ascendc-tiling-design` | 多核切分、UB 切分、Buffer 规划 | Architect / Developer / Strategy |
| `ascendc-npu-arch` | 芯片、架构、SocVersion、条件编译 | Architect / Verifier |
| `ascendc-env-check` | NPU 和 CANN 环境检查 | Tester / Verifier |
| `ops-precision-standard` | dtype 阈值、精度验收规则 | Tester / Reviewer |
| `ascendc-precision-debug` | 正确性失败诊断 | Repair / Memory |
| `ascendc-runtime-debug` | 运行时错误、plog、错误码 | Repair / Memory |
| `ops-profiling` | msprof、性能指标、瓶颈分析 | Tester / Strategy |
| `ascendc-code-review` | 静态审查和规则检查 | Reviewer |

同时保留主干已有增强：

| Skill | 作用 |
| --- | --- |
| `ascendc-msopgen-workflow` | custom operator / msopgen 实战流程与坑点 |
| `ascendc-kb-docs` | Ascend C 深度知识库索引 |
| `ascendc-kernel-*` | direct invoke 相关角色 |
| `ascendc-ops-*` | custom operator 相关角色 |
| `ops-direct-invoke-team` | direct invoke team 工作流 |

## 4. 暂不补充的 Skills

Baseline v0 暂不引入这些缺失 skill，避免路线发散：

| Skill | 暂缓原因 |
| --- | --- |
| `torch-ascendc-op-extension` | direct invoke 接 PyTorch 的后续路线，v0 先走 custom operator |
| `ascendc-docs-gen` | 文档生成可后置到 Reporter |
| `ascendc-performance-best-practices` | 先以 `ops-profiling` 为性能分析核心 |
| `ascendc-regbase-best-practice` | 仅在 Ascend950/arch35 成为目标时引入 |
| PyPTO 系列 | 非 v0 主线 |
| model inference 系列 | 目标不同 |

## 5. 基线工作流

Baseline v0 的标准闭环：

```text
1. Operator Spec
   -> workspace/specs/{op}.md

2. Architect
   -> 读取 spec、lineage、score、memory
   -> 检索 CANN docs / skills
   -> 输出 DESIGN.md、PLAN.md、strategy_id

3. Developer
   -> 生成或修改 custom operator 工程
   -> op_host / op_kernel / CppExtension / reference

4. Reviewer
   -> 静态审查
   -> API、tiling、precision、performance risk
   -> 输出 REVIEW.md / review summary

5. Tester / Scoring
   -> scoring/score.sh
   -> preflight
   -> compile
   -> deploy
   -> pybind
   -> correctness
   -> performance
   -> score aggregation

6. Evolution Manager
   -> correctness pass + performance improved: promote to best
   -> otherwise: reject attempt, retain failure summary

7. Supervisor
   -> stall / repeated failure / repeated direction
   -> redirect strategy

8. Reporter
   -> 汇总 lineage、score、logs、经验
```

## 6. 当前主干目录约定

沿用主干代码的结构：

```text
agents/
  AGENTS.md
  architect/
  developer/
  reviewer/
  tester/
  supervisor/
  reporter/

scoring/
  score.sh
  compile.sh
  deploy.sh
  build_pybind.sh
  test_correctness.py
  test_performance.py
  compute_score.py
  configs/

evolution/
  config.yaml
  state.json
  scores/
  logs/
  redirects/

workspace/
  specs/
  runs/{op_name}/
    best/
    attempts/step_N/
    test/
  deploy/opp/

Knowledge-base/
.claude/skills/
```

## 7. Baseline v0 新增目录建议

后续在主干中新增这些目录：

```text
verifier/
  summarize_compile_log.py
  summarize_runtime_log.py
  summarize_correctness.py
  summarize_profiling.py

memory/
  long_term/
  short_term/
  experience/
  strategy_pool/

dataset_export/
  export_attempt.py
  export_preference_pair.py
```

v0 可以先只建 skeleton，不急着实现复杂 value update。

## 8. Verifier 基线

当前 `scoring/score.sh` 是 Baseline v0 的 verifier 核心。

已有阶段：

```text
environment/preflight
compile
deploy
pybind
correctness
performance
score aggregation
```

v0 改造目标：

每个阶段都输出一个结构化 summary：

```text
evolution/logs/step_N/
  preflight.log
  compile.log
  deploy.log
  pybind.log
  correctness.log
  performance.log
  compile_summary.json
  runtime_summary.json
  correctness_summary.json
  performance_summary.json
```

`evolution/scores/vN.json` 引用这些 summary，并统一记录：

```yaml
failure_type:
error_signature:
correctness_total:
performance_total:
improvement_over_best:
test_levels_run:
phase_timings:
strategy_id:
```

## 9. Memory Baseline

Baseline v0 先采用双层 memory skeleton：

### Long-term Memory

来源：

- CANN skills。
- API rules。
- tiling patterns。
- precision rules。
- runtime error rules。
- profiling bottleneck rules。

建议路径：

```text
memory/long_term/
  api_rules.yaml
  tiling_patterns.yaml
  precision_rules.yaml
  runtime_errors.yaml
  profiling_bottlenecks.yaml
```

### Short-term Memory

来源：

- 当前算子的 attempts。
- 编译错误。
- 正确性失败。
- profiling 结果。
- 已尝试 strategy。
- 不应重复方向。

建议路径：

```text
workspace/runs/{op_name}/memory/
  short_term.md
  strategy_attempts.json
  failure_clusters.json
  do_not_repeat.md
```

v0 不强制实现 Q-value，但预留字段。

## 10. Strategy Pool Baseline

第一批策略：

| Strategy ID | 触发瓶颈 | 思路 |
| --- | --- | --- |
| `mte2_datacopy_granularity` | MTE2 bound | 增大搬运粒度，非对齐使用 DataCopyPad |
| `vec_reduce_cast` | VEC bound | 减少 Cast，控制 FP32 中间计算开销 |
| `double_buffer_pipeline` | MTE/VEC overlap 低 | 检查 TQue/TPipe、EnQue/DeQue、双缓冲 |
| `core_balance_tiling` | 核间不均衡 | 调整 blockDim、尾核和切分维度 |
| `ub_buffer_reuse` | UB pressure | 复用中间 buffer，减少 Alloc/Free 和 UB 占用 |
| `bank_conflict_padding` | bank conflict | 调整 UB layout 或 padding |

每个 attempt 必须记录：

```yaml
strategy_id:
strategy_reason:
expected_effect:
actual_result:
```

## 11. Lineage Baseline

沿用：

```text
workspace/runs/{op_name}/best/
workspace/runs/{op_name}/attempts/step_N/
evolution/scores/vN.json
evolution/logs/step_N/
```

v0 增加 lineage summary：

```text
workspace/runs/{op_name}/lineage.md
```

记录：

- accepted versions。
- rejected attempts。
- failure clusters。
- best score history。
- strategy history。
- supervisor redirects。

## 12. Dataset Export Baseline

v0 先定义导出格式，不急着训练：

```json
{
  "op_name": "gelu_custom",
  "attempt_id": "step_3",
  "spec": "...",
  "retrieved_knowledge": [],
  "strategy_id": "double_buffer_pipeline",
  "plan": "...",
  "code_diff": "...",
  "compile_summary": {},
  "correctness_summary": {},
  "performance_summary": {},
  "score": {},
  "outcome": "accepted|rejected",
  "experience_summary": "..."
}
```

目标：

- SFT 样本。
- repair 样本。
- optimization 样本。
- preference pair。
- process reward 样本。

## 13. Baseline v0 验收标准

最低验收：

- 能选择一个算子 spec，例如 `gelu_custom`。
- 能创建一次 attempt。
- 能运行 `scoring/score.sh`，即使失败也能产生 `vN.json`。
- failure_type 明确。
- logs 留存完整。
- attempt 和 score 可以被 lineage 读取。

增强验收：

- 编译失败可生成 `compile_summary.json`。
- correctness 失败可生成 `correctness_summary.json`。
- performance 成功可生成 `performance_summary.json`。
- 每轮 attempt 有 `strategy_id`。
- short-term memory 能记录不应重复的失败方向。

## 14. 下一步实现顺序

1. 编写 skill 差异审计脚本：比较主干内置 skill 与 `cann/skills` 上游。
2. 在 `scoring/` 增加 summary 生成脚本。
3. 修改 `score.sh`，让每个阶段写 JSON summary。
4. 增加 `strategy_id` 字段并贯穿 PLAN、score、lineage。
5. 建立 memory skeleton。
6. 实现 attempt dataset export。

## 15. 当前结论

Baseline v0 不需要再“引入 10 个核心 skill”，因为根目录主干已经包含。当前真正要做的是：

```text
确认内置 skill 版本
  -> 做上游差异审计
  -> 保留 AscendC-Kernel-Agent 的本地增强
  -> 改造 scoring/verifier 结构化输出
  -> 增加 memory、strategy、dataset skeleton
```
