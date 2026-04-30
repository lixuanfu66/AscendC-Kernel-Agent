# Kernel 编程 Agent 的 Memory 与自进化机制深度总结

分析对象：

- AutoKernel: Autonomous GPU Kernel Optimization via Iterative Agent-Driven Search
- AVO: Agentic Variation Operators for Autonomous Evolutionary Search
- EvoKernel: Towards Cold-Start Drafting and Continual Refining
- CUDA Agent: Large-Scale Agentic RL for High-Performance CUDA Kernel Generation
- KernelSkill / KernelMem: A Multi-Agent Framework for GPU Kernel Optimization
- cuPilot: A Strategy-Coordinated Multi-agent Framework for CUDA Kernel Evolution

## 总体判断

这 6 条线可以归纳为 6 种互补能力：

| 系统 | 最值得吸收的能力 |
| --- | --- |
| AutoKernel | 工程化 edit-benchmark loop、Amdahl 调度、严格 correctness harness |
| AVO | 把 Agent 变成 evolution operator，维护 lineage 并持续进化 |
| EvoKernel | value-driven memory、阶段性 Q-value、NPU 冷启动和持续优化 |
| CUDA Agent | 大规模 agentic RL、skill-augmented environment、训练数据合成 |
| KernelSkill / KernelMem | 双层 memory：长期专家技能 + 短期 anti-backtracking |
| cuPilot | strategy 作为中间表示、roofline-guided prompting、strategy pool |

对 CANN 算子 Agent 来说，最佳组合不是照搬某一篇，而是构造：

```text
CANN Agent = AutoKernel verifier
           + AVO lineage evolution
           + EvoKernel value memory
           + KernelSkill long/short memory
           + cuPilot strategy pool
           + CUDA Agent data/RL pipeline
```

## 1. AutoKernel：工程闭环和验证体系

### 核心技术

AutoKernel 关注任意 PyTorch 模型中的 GPU kernel 优化。它先 profiling 模型，找出瓶颈，再按 Amdahl 定律估计每个 kernel 的全局收益，优先优化对端到端性能影响最大的部分。

关键机制：

- 模型级 profiling，而不是孤立 kernel 优化。
- Amdahl impact 排序，避免优化收益很小的 kernel。
- Triton / CUDA C++ 双后端。
- 单文件小步 edit，benchmark 后 keep 或 revert。
- 五阶段 correctness harness：
  - smoke tests
  - shape sweeps
  - numerical stability
  - determinism
  - edge cases
- 六层优化 playbook：从 block size、memory access 到 architecture-specific 和 kernel-specific。

### 有价值方案

对 CANN 项目，AutoKernel 的最大价值是“把 Agent 约束在可验证工程闭环里”。

建议迁移：

```text
spec -> generate -> compile -> smoke correctness
     -> shape sweep -> numerical tolerance -> determinism
     -> boundary cases -> performance -> keep/revert
```

每次尝试都生成一个 `attempt`，只有通过 correctness 后才允许进入 performance scoring。

### 对 CANN 的改造点

- Amdahl 调度改成 CANN operator / model profiling 调度。
- Triton/CUDA backend 改为 Ascend C direct invoke 和 msopgen custom op 两种 backend。
- correctness harness 对接 PyTorch/torch_npu reference。
- performance 对接 NPU Event timing + msprof。
- playbook 改成 CANN 版：tiling、DataCopyPad、TPipe/TQue、double buffer、UB/L1/L0、Vector/Cube、核间负载均衡。

## 2. AVO：Agentic Variation Operator 与 Lineage

### 核心技术

AVO 的关键不是“用 LLM 生成候选代码”，而是把进化搜索中的 variation operator 本身替换成自治 coding agent。

传统进化：

```text
candidate' = mutation_or_crossover(sample(population))
```

AVO：

```text
candidate' = Agent(P_t, K, f)
```

其中：

- `P_t`：历史 lineage，包含版本、分数、失败轨迹。
- `K`：领域知识库。
- `f`：编译、正确性和性能反馈。

Agent 可以检索、编辑、修复、批评、验证，并基于当前 lineage 主动选择方向。

### 有价值方案

AVO 最大价值是明确了：高性能 kernel 搜索中，“版本谱系”本身就是 memory。

建议迁移到 CANN：

```yaml
lineage:
  accepted_versions:
    - version:
      code_ref:
      score:
      performance:
      key_strategy:
      profiling_summary:
  rejected_attempts:
    - attempt:
      failure_type:
      error_signature:
      strategy:
      reason:
  active_hypotheses:
    - hypothesis:
      evidence:
      status:
```

### 对 CANN 的改造点

在 `AscendC-Kernel-Agent` 的 `best/`、`attempts/`、`evolution/scores/` 基础上，补充：

- lineage summary 自动生成。
- rejected attempts 保留压缩摘要。
- 每个版本必须标记 strategy。
- Supervisor 不只看 stall，也分析 direction repeat、failure cluster、performance plateau。

## 3. EvoKernel：Value-Driven Memory 与阶段性 Q-value

### 核心技术

EvoKernel 面向 NPU/Ascend C 这种数据稀缺生态，核心是把 kernel synthesis 建模为 memory-based RL。

它把任务分成两阶段：

- cold-start drafting：先得到可编译、可正确运行的 kernel。
- continual refining：再优化 latency。

记忆检索不是只按语义相似度，而是维护阶段性价值：

```text
Q1(memory) = 对生成可行 kernel 的贡献
Q2(memory) = 对优化 latency 的贡献
```

验证器反馈会更新 memory value。跨任务 memory sharing 让简单算子的经验迁移到复杂算子。

### 有价值方案

EvoKernel 是最贴近我们 CANN 项目的方案。它告诉我们经验库不能只是 markdown 笔记，而应是可评分、可更新、可参与决策的 memory system。

建议 schema：

```yaml
memory_id:
content:
operator_tags:
failure_tags:
strategy_tags:
stage_values:
  compile_q:
  correctness_q:
  performance_q:
  reuse_q:
evidence:
  attempts:
  score_json:
  profiling:
last_used:
success_count:
failure_count:
applicability:
confidence:
```

### 对 CANN 的改造点

- 把 `score.sh` 的每次结果变成 memory 更新信号。
- 编译失败时更新 `compile_q`。
- 正确性失败时更新 `correctness_q`。
- 性能提升时更新 `performance_q`。
- 被多个算子复用成功时更新 `reuse_q`。

最重要的是阶段性检索：

```text
seed generation -> 优先检索 compile_q / correctness_q 高的 memory
repair          -> 优先检索相同 failure_signature 的 memory
perf tuning     -> 优先检索 performance_q 高且瓶颈类型匹配的 memory
```

## 4. CUDA Agent：Agentic RL 与 Skill-Augmented Environment

### 核心技术

CUDA Agent 走的是“让模型内化 CUDA 优化能力”的路线。它包含三件事：

- scalable data synthesis pipeline。
- skill-augmented CUDA development environment。
- reinforcement learning algorithmic techniques，用于稳定长上下文训练。

环境提供自动 verification 和 profiling，给 RL 提供可靠 reward。它不是只靠推理时反复修，而是通过 agentic RL 提升模型自身的 CUDA kernel 能力。

### 有价值方案

CUDA Agent 对我们最有价值的是“训练闭环设计”：

```text
task synthesis
  -> agent rollout
  -> compile/correctness/perf verifier
  -> reward
  -> trajectory dataset
  -> RL/SFT/DPO training
  -> stronger model
  -> better rollout
```

### 对 CANN 的改造点

构建 CANN 版 skill-augmented environment：

- skills：CANN docs search、API best practice、tiling、profiling、precision debug。
- tools：msopgen、build、run、torch_npu correctness、NPU Event、msprof。
- reward：
  - compile pass
  - correctness pass
  - latency improvement
  - fewer iterations
  - valid experience summary

数据产物：

```yaml
trajectory:
  spec:
  retrieved_skills:
  plan:
  code_diff:
  compile_result:
  correctness_result:
  performance_result:
  reward:
  final_experience:
```

### 现实注意

CUDA Agent 更像训练体系，不是第一阶段最小原型。我们应先用 `AscendC-Kernel-Agent + value memory` 收集轨迹，再考虑 RL。

## 5. KernelSkill / KernelMem：双层 Memory

### 核心技术

KernelSkill 认为现有 LLM kernel pipeline 依赖模型内部隐式启发式，导致试错低效且不可解释。它用专家优化技能替代隐式 heuristic，并提出双层 memory：

- long-term memory：可复用专家技能、硬件瓶颈知识、kernel 结构知识。
- short-term memory：当前任务的历史 kernel、错误日志、profiling、修复记录，用于防止重复回退。

开源 KernelMem 中，`memorybank/` 保存长期知识；每轮生成的 code、evaluation、profile、optimization_tree 会作为短期 memory 注入后续 prompt。

### 有价值方案

这是我们最容易立刻实现的 memory 设计。

CANN 版双层 memory：

```text
Long-term memory:
  CANN API 使用规则
  tiling 模式
  DataCopy/UB/Queue/DoubleBuffer 最佳实践
  常见编译错误和修复
  常见性能瓶颈和优化策略

Short-term memory:
  当前算子的 attempts
  最近错误日志
  最近 profiling
  已尝试策略
  不应重复方向
```

### 对 CANN 的改造点

在 `AscendC-Kernel-Agent` 中添加：

- `experience/long_term/`
- `workspace/runs/{op}/memory/short_term.md`
- `optimization_tree.json`
- `do_not_repeat.md`
- `strategy_attempts.json`

短期 memory 要重点记录失败：

```yaml
attempt_id:
strategy:
failure_type:
error_signature:
root_cause:
next_time_avoid:
```

## 6. cuPilot：Strategy 作为中间表示

### 核心技术

cuPilot 认为 LLM + evolution 失败的重要原因是“进化表示错位”：直接对代码做 crossover/mutation 太难，长链推理不稳定。它引入 strategy 作为中间语义表示。

核心机制：

- Strategy-Coordinated Evolution。
- strategy-level crossover，再做 strategy-to-kernel translation。
- roofline-guided prompting：先判断 compute-bound、memory-bound 或中间态。
- strategy-level population initialization：从历史数据和 RAG 中建立 strategy pool。

### 有价值方案

cuPilot 对 CANN 最大价值是：经验库中不只存“代码片段”和“错误修复”，还要存“优化策略”。

CANN strategy schema：

```yaml
strategy_id:
name:
operator_pattern:
bottleneck_type: vec_bound | mte2_bound | cube_bound | scalar_bound | imbalance | bank_conflict
preconditions:
implementation_steps:
expected_effect:
risk:
verification:
examples:
value:
```

### 对 CANN 的改造点

把性能优化改成两步：

```text
profiling -> bottleneck classification -> strategy selection/generation -> code edit
```

不要让 Agent 直接“优化代码”。先让它选择策略，例如：

- MTE2 bound：增大 DataCopy 粒度、检查对齐、启用 double buffer。
- VEC bound：融合指令、减少 cast、改用向量 API。
- 核间不均衡：调整 tiling 切分和尾核处理。
- bank conflict：调整 UB layout 或 padding。

## 横向比较

| 维度 | AutoKernel | AVO | EvoKernel | CUDA Agent | KernelSkill | cuPilot |
| --- | --- | --- | --- | --- | --- | --- |
| 主要贡献 | 工程闭环 | Agentic evolution | Value memory | RL 训练闭环 | 双层 memory | Strategy evolution |
| Memory | playbook/history | lineage | Q-value memory | skill/env/trajectory | long + short | strategy pool |
| Evolution | iterative search | variation operator | self-evolving RL | agentic RL rollout | multi-round memory loop | strategy-coordinated |
| Verifier | 最强 correctness harness | execution feedback | multi-gate verifier | verification + profiling reward | compile/run/profile | benchmark + roofline |
| 对 CANN 价值 | 高 | 高 | 极高 | 高但偏中长期 | 极高 | 极高 |

## 推荐的 CANN Agent 架构

```text
1. Task Spec
   -> 解析算子规格、shape/dtype/layout、target chip

2. Stage Controller
   -> seed / repair / optimize / report

3. Retrieval
   -> long-term skill memory
   -> short-term attempt memory
   -> value-aware experience memory
   -> strategy pool

4. Planner
   -> 先选 strategy，再写 PLAN

5. Developer
   -> 生成或修改 Ascend C / custom op 工程

6. Verifier
   -> environment
   -> compile
   -> deploy
   -> pybind/direct invoke
   -> smoke correctness
   -> boundary correctness
   -> representative correctness
   -> performance
   -> profiling

7. Evolution Manager
   -> keep/reject
   -> update lineage
   -> update Q-values
   -> update strategy scores
   -> trigger supervisor redirect

8. Data Export
   -> experience entries
   -> trajectory dataset
   -> preference pairs
   -> process reward samples
```

## 最值得立即实现的 5 个模块

### 1. CANN Verifier

基于 `AscendC-Kernel-Agent/scoring/score.sh` 扩展：

- 增加 anti-hacking/constraint check。
- 编译、部署、正确性、性能分层早停。
- 正确性通过前不记录性能提升。

### 2. Long/Short Memory

从 KernelSkill 借鉴：

- long-term：CANN skill + 经验库。
- short-term：当前任务 attempts、错误、profile、已尝试策略。

### 3. Value-aware Memory

从 EvoKernel 借鉴：

- `compile_q`
- `correctness_q`
- `performance_q`
- `reuse_q`

### 4. Strategy Pool

从 cuPilot 借鉴：

- 为每个优化策略建立结构化条目。
- 根据 profiling bottleneck 检索策略。
- 先生成 strategy，再生成代码。

### 5. Lineage Manager

从 AVO 借鉴：

- accepted lineage。
- rejected lineage。
- failure cluster。
- improvement plateau。
- supervisor redirect。

## 对训练语料的启发

每次 attempt 都应导出多种训练样本：

```text
SFT:
  spec + memory + plan -> code

Repair:
  code + compile/runtime/correctness error + memory -> fixed code

Optimization:
  code + profiling + strategy pool -> optimized code

Preference:
  attempt A vs attempt B -> better trajectory

Process reward:
  state + action -> stage reward
```

## 一句话方案

本项目最有价值的技术路线是：以 `AscendC-Kernel-Agent` 的 scoring 和 lineage 为骨架，吸收 KernelSkill 的双层记忆、EvoKernel 的阶段性 Q-value、cuPilot 的 strategy pool、AutoKernel 的 verifier、AVO 的 Agentic variation，再把轨迹导出为 CUDA Agent 式 RL/训练数据。

