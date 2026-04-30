# CUDA/NPU 算子编程 Agent：Memory 与自进化机制持续跟踪策划

## 目标

建立一个长期技术雷达，持续跟踪 CUDA、Ascend C、Triton、AMD/HIP、其他 NPU/AI 加速器方向的算子编程 Agent，重点关注：

- memory / experience / skill 管理机制。
- self-evolution / evolutionary search / RL 机制。
- 编译、正确性、性能 profiling 的 verifier 设计。
- benchmark、训练数据、开源工程、开源 skill。
- 对 CANN 算子编程 Agent 的可迁移设计。

## 跟踪问题

1. 新论文提出了什么 memory 机制：短期记忆、长期经验库、skill memory、value-aware memory、lineage memory、strategy pool。
2. 新论文如何做自进化：迭代修复、进化搜索、population、mutation/crossover、agentic variation、RL、self-play。
3. verifier 如何设计：anti-hacking、编译、正确性、性能、profiling、stress test。
4. 经验如何更新：是否有 reward、Q-value、置信度、success/failure attribution。
5. 是否开源代码、数据集、skill、prompt、benchmark。
6. 对 Ascend C/CANN 的直接可迁移点是什么。

## 关键词矩阵

### 核心主题关键词

```text
LLM kernel optimization agent
GPU kernel generation agent
CUDA kernel optimization agent
CUDA kernel generation reinforcement learning
NPU kernel synthesis agent
Ascend C kernel generation
Triton kernel generation agent
agentic kernel optimization
kernel evolution agent
autonomous kernel optimization
```

### Memory / Experience 关键词

```text
kernel agent memory
value-driven memory kernel synthesis
experience memory CUDA kernel
skill memory kernel optimization
dual-level memory kernel optimization
long-term memory short-term memory GPU kernel
trajectory-aware memory kernel agent
lineage memory kernel evolution
strategy pool kernel evolution
RAG kernel optimization agent
```

### 自进化 / RL 关键词

```text
agentic evolutionary search kernel
evolutionary CUDA kernel optimization
self-evolving kernel agent
agentic variation operators
strategy coordinated evolution CUDA
kernel optimization reinforcement learning
agentic RL CUDA kernel generation
population based kernel optimization LLM
roofline guided prompting CUDA kernel
hardware feedback kernel optimization agent
```

### Verifier / Benchmark 关键词

```text
KernelBench agent
NPUKernelBench
ComputeEval CUDA
CUDA benchmark LLM code generation
kernel correctness verification LLM
anti-hacking kernel generation
profiling guided kernel optimization
hardware feedback CUDA agent
msprof agent kernel optimization
torch.compile baseline kernel generation
```

### 开源工程 / Skill 关键词

```text
GitHub CUDA kernel agent
GitHub KernelBench agent
GitHub Triton kernel generation agent
GitHub kernel optimization SKILL.md
CUDA Agent SKILL.md
KernelSkill KernelMem GitHub
AKO Agentic Kernel Optimization
OpenEvolve kernel optimization
EvoKernel CUDA GitHub
CANN skills Ascend C agent
```

## 重点跟踪对象初表

| 名称 | 类型 | 关注点 | 状态 |
| --- | --- | --- | --- |
| CUDA Agent | 论文/项目/数据集/skill | 大规模 agentic RL、skill-augmented environment、CUDA-Agent-Ops-6K | 高优先级 |
| EvoKernel / Value-Driven Memory | 论文 | NPU/Ascend C、value-driven memory、阶段性 Q-value | 高优先级 |
| KernelSkill / KernelMem | 论文/代码 | dual-level memory、long-term expert skills、short-term anti-backtracking | 高优先级 |
| AVO | 论文/方法 | agentic variation operator、lineage-aware evolution | 高优先级 |
| AutoKernel | 论文/代码 | profiling + Amdahl 调度 + edit-benchmark loop | 高优先级 |
| cuPilot | 论文/代码 | strategy 作为中间表示、roofline-guided prompting、strategy pool | 高优先级 |
| CudaForge | 论文 | hardware feedback、training-free multi-agent optimization | 中高优先级 |
| Astra | 论文/代码 | multi-agent CUDA kernel optimization、profiling/planning/refinement | 中高优先级 |
| KernelAgent / PyTorch KernelFalcon | 开源工程 | PyTorch program 到 Triton kernel，硬件引导优化 | 中高优先级 |
| ComputeEval | benchmark | CUDA 代码能力评测，功能正确性测试 | 中优先级 |
| awesome-LLM-driven-kernel-generation | awesome list | 新论文和项目入口 | 持续跟踪 |
| cann/skills | skill 仓库 | CANN 官方/社区 skill、profiling、debug、docs search | 持续同步 |
| AscendC-Kernel-Agent | 开源原型 | Ascend C agent、scoring、lineage、evolution | 本项目主干 |

## 信息源

### 每周检索

- arXiv：`cs.LG`、`cs.AI`、`cs.DC`、`cs.PL`、`cs.PF`。
- OpenReview：ICLR、NeurIPS、MLSys、OSDI/SOSP/ASPLOS/ISCA 相关投稿。
- GitHub Search：关键词 + recently updated。
- Hugging Face Papers。
- Papers with Code / CatalyzeX / hgpu.org / EmergentMind。
- GitCode / Gitee：CANN、Ascend C、NPU 相关仓库。

### 每月检索

- NVIDIA Developer Blog。
- PyTorch blog / meta-pytorch projects。
- AMD ROCm/HIP blog。
- 华为昇腾社区、CANN 文档、CANN GitCode。
- KernelBench、MultiKernelBench、NPUKernelBench、ComputeEval 等 benchmark 更新。

## 检索节奏

### 每周快照，建议周一

目标：发现新论文、新 repo、新 release。

输出：

- 新增条目列表。
- 每条 3-5 行摘要。
- 是否值得深读。
- 是否值得进入本项目知识库。

### 每月深度分析

目标：对高优先级条目做技术拆解。

输出：

- 技术机制卡片。
- memory/evolution/verifier 对比表。
- 对 CANN 项目的迁移建议。
- 需要实现的实验点。

### 每季度综述

目标：形成阶段性趋势判断。

输出：

- 技术路线图更新。
- 代表性系统横向比较。
- 本项目架构是否需要调整。
- 候选 benchmark 和训练数据计划。

## 筛选标准

### 高优先级

满足任意两项：

- 明确提出 memory / experience / skill / value / lineage / strategy pool。
- 有自进化、RL、population、agentic variation 或持续优化机制。
- 有开源代码、数据集、skill 或 benchmark。
- 覆盖 CUDA/Triton/Ascend C/NPU kernel。
- 有可复现实验，包含 correctness 和 performance。

### 中优先级

- 只提供框架或论文，无代码。
- 只做通用 GPU codegen，但有 verifier 或 profiling 设计。
- 只做 benchmark，但可用于评测。

### 低优先级

- 只讨论通用 coding agent。
- 没有 kernel/hardware/profiling 细节。
- 没有可复现评价或只做概念演示。

## 每篇论文/项目记录模板

```yaml
id:
title:
url:
date_found:
publish_date:
type: paper | repo | skill | benchmark | dataset | blog
hardware: CUDA | Triton | Ascend C | HIP | NPU | TPU | other
code_available:
dataset_available:
benchmark:
memory_mechanism:
evolution_mechanism:
verifier:
reward_or_score:
profiling_feedback:
open_source_assets:
key_results:
limitations:
relevance_to_cann_agent:
follow_up_actions:
status: new | triaged | deep_read | integrated | archived
```

## 技术标签体系

### Memory 标签

- `semantic-rag`
- `short-term-memory`
- `long-term-memory`
- `skill-memory`
- `experience-memory`
- `lineage-memory`
- `strategy-pool`
- `value-aware-memory`
- `q-value-memory`
- `failure-memory`

### Evolution 标签

- `edit-test-loop`
- `agentic-variation`
- `population-search`
- `strategy-crossover`
- `mutation`
- `self-play`
- `agentic-rl`
- `offline-rl`
- `preference-optimization`
- `supervisor-redirect`

### Verifier 标签

- `anti-hacking`
- `compile-check`
- `correctness-check`
- `numerical-tolerance`
- `performance-check`
- `profiling-feedback`
- `stress-test`
- `hardware-counter`

## 每周检索 Query 模板

```text
("CUDA kernel" OR "GPU kernel" OR "Triton kernel") ("agent" OR "LLM") ("memory" OR "experience" OR "skill" OR "evolution" OR "reinforcement learning")
("NPU kernel" OR "Ascend C" OR "CANN") ("agent" OR "LLM") ("memory" OR "self-evolving" OR "synthesis")
("KernelBench" OR "NPUKernelBench" OR "ComputeEval") ("agent" OR "memory" OR "evolution")
("kernel optimization") ("agentic variation" OR "evolutionary search" OR "strategy pool" OR "roofline guided")
site:github.com ("CUDA kernel" "agent" "SKILL.md")
site:github.com ("KernelBench" "agent" "memory")
site:github.com ("Triton" "kernel" "agent" "optimization")
site:gitcode.com ("CANN" OR "Ascend C") ("skill" OR "agent")
```

## 产物目录建议

```text
research_tracking/
  README.md
  weekly/
    2026-W18.md
  monthly/
    2026-04.md
  papers/
    cuda-agent.md
    kernelskill.md
    cupilot.md
  repos/
    kernelagent.md
    kernelmem.md
  assets/
    query_templates.md
```

## 自动化建议

第一阶段先人工/半自动：

1. 每周运行固定 query。
2. 把候选条目写入 `research_tracking/weekly/YYYY-WW.md`。
3. 对高优先级条目生成 `papers/` 或 `repos/` 卡片。
4. 每月把条目汇总到 `findings.md`。

第二阶段再自动化：

1. 脚本抓取 arXiv、GitHub、Hugging Face Papers、OpenReview。
2. 自动去重：按 title、arXiv id、GitHub URL。
3. 自动打标签：memory/evolution/verifier/hardware。
4. 自动生成候选摘要。
5. 人工确认后进入知识库。

## 当前已发现的代表性来源

- CUDA Agent: Large-Scale Agentic RL for High-Performance CUDA Kernel Generation
- KernelSkill: A Multi-Agent Framework for GPU Kernel Optimization
- CudaForge: An Agent Framework with Hardware Feedback for CUDA Kernel Optimization
- cuPilot: A Strategy-Coordinated Multi-agent Framework for CUDA Kernel Evolution
- KernelAgent / PyTorch KernelFalcon
- ComputeEval
- awesome-LLM-driven-kernel-generation
- EvoKernel / Value-Driven Memory for NPU Kernel Synthesis
- cann/skills
- AscendC-Kernel-Agent

