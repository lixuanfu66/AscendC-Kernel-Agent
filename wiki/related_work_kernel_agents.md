# 相关论文调研：Kernel 编程 Agent 与进化优化

## AutoKernel: Autonomous GPU Kernel Optimization via Iterative Agent-Driven Search

来源：arXiv:2603.21331，2026-03-22。

核心问题：自动优化 PyTorch 模型中的 GPU kernel，减少人工 kernel 工程调优成本。

关键技术：

- 模型级 profiling：先对完整 PyTorch 模型做 profiling，而不是孤立优化单个 kernel。
- Amdahl 定律调度：按 kernel 对总耗时的贡献分配优化预算。
- 单文件 edit-benchmark-keep/revert 闭环：Agent 修改一个 kernel 文件，benchmark 通过则保留，否则回滚。
- 五阶段正确性验证：smoke test、shape sweep、数值稳定性、确定性、边界 case；全部通过后才测性能。
- 双后端：Triton 用于快速迭代，CUDA C++ 用于底层控制。
- 六层优化 playbook：block size、memory access、compute、advanced、architecture-specific、kernel-specific。

对 CANN 算子 Agent 的启发：

- 需要先做“算子/模型级瓶颈排序”，不能平均用力。
- 编译与正确性验证要作为硬门禁，性能数据只在正确后采信。
- 单次修改范围应尽量小，方便归因和回滚。
- 优化经验可以先做成 Agent-readable playbook，再逐步演化为经验库。

## AVO: Agentic Variation Operators for Autonomous Evolutionary Search

来源：arXiv:2603.24517，2026-03-25。

核心问题：把进化搜索中的变异算子从固定 mutation/crossover/heuristics 升级为自治 coding agent。

关键技术：

- Agentic Variation Operator：用 `Agent(P_t, K, f)` 替代传统 `Generate(Sample(P_t))`。
- Lineage-aware search：Agent 可以查看历史版本、性能分数和 profiling 特征，选择要继承或重访的方向。
- Knowledge-base-aware variation：Agent 使用 CUDA/PTX/Blackwell 文档和现有 attention kernel 实现。
- 内部 edit-evaluate-diagnose 循环：一个 variation step 内部可能包含多次编辑、测试、修复和策略调整。
- Continuous evolution：长时间无人值守进化，每个有效改进作为 git commit 和分数进入 lineage。
- Self-supervision intervention：当搜索停滞或陷入无效循环时，由监督机制回顾轨迹并重定向探索。

实验要点：

- 在 NVIDIA B200 上优化 attention kernel。
- 7 天连续进化，产生 40 个 committed 版本，内部探索超过 500 个候选方向。
- MHA 上最高超过 cuDNN 3.5%、超过 FlashAttention-4 10.5%。
- MHA 进化出的优化可在约 30 分钟内迁移到 GQA。

对 CANN 算子 Agent 的启发：

- 经验库不只是检索材料，也可以作为“进化 lineage”参与下一轮 variation。
- 需要保存失败尝试，即使不进入 committed lineage，也应进入可分析轨迹。
- 对高性能算子，收益往往来自离散架构级跃迁和后续微架构级细调。
- 可设计 CANN 版 `Agent(P_t, K, f)`：`P_t` 是历史算子版本，`K` 是 CANN/Ascend C 知识库，`f` 是编译、正确性、性能评分函数。

## Towards Cold-Start Drafting and Continual Refining: A Value-Driven Memory Approach with Application to NPU Kernel Synthesis

来源：arXiv:2603.10846，2026-03-11。论文系统名为 EvoKernel。

核心问题：在 NPU/Ascend C 这种数据稀缺生态中，不依赖昂贵微调，让通用 LLM 通过记忆和反馈完成 kernel synthesis。

关键技术：

- Memory-based MDP：把 kernel synthesis 建模为带动态记忆库的强化学习问题。
- 两阶段流程：
  - Cold-start drafting：先生成可编译、可正确运行的初始 kernel。
  - Continual refining：在可行 kernel 基础上继续优化 latency。
- Value-Driven Retrieval：不是只按语义相似度检索，而是学习记忆条目的阶段性 Q-value。
- 阶段性 Q 值：
  - `Q1` 衡量记忆对生成可行 kernel 的贡献。
  - `Q2` 衡量记忆对 latency 优化的贡献。
- 异构记忆库：API templates、成功/失败经验、generation traces、best practices。
- 价值更新：使用 verifier 反馈更新被检索记忆项的 Q 值。
- 多门验证器：anti-hacking、compilation、correctness、latency，Ascend C 场景使用对应 toolchain 和 profiling。
- 跨任务共享：简单算子的经验可迁移到复杂算子，强模型生成的 memory 也能辅助弱模型。

实验要点：

- 构建 NPU 版 KernelBench，并主要实例化在 Ascend C。
- GPT-5.2 上整体 Acc 从 4.0% 提升到 83.0%，CR 从 11.0% 提升到 98.5%。
- 迭代优化相对初始正确版本的 median speedup 为 3.60x。
- L1 到 L2 的经验迁移显著优于 L2 从零开始。
- GPT-5.2 构建的 memory 可迁移给 DeepSeek/Qwen 等较弱模型，显著提升编译率和正确率。

对 CANN 算子 Agent 的启发：

- 我们项目可以直接采用“两阶段目标”：先正确，再优化性能。
- 经验库应包含成功和失败，并为不同阶段维护不同价值分数。
- 检索策略应从 similarity-only 升级为 value-aware retrieval。
- 训练语料可从 memory 更新过程自然产生：检索项、代码、验证结果、reward、Q-value 更新。

## 三篇论文的横向比较

| 维度 | AutoKernel | AVO | EvoKernel |
| --- | --- | --- | --- |
| 主要目标 | GPU kernel 自动优化 | 进化搜索算子 agent 化 | NPU/Ascend C 冷启动生成与持续优化 |
| 核心闭环 | edit-benchmark-keep/revert | Agent 作为 variation operator | value-driven memory + verifier feedback |
| 记忆/知识 | playbook、history、profiling | lineage、知识库、执行反馈 | 动态记忆库、Q-value、跨任务经验 |
| 评估函数 | 正确性门禁 + 性能 | 正确性 + TFLOPS | anti-hacking + 编译 + 正确性 + latency |
| 搜索组织 | Amdahl 排序，多 kernel 调度 | 单 lineage 连续进化 | cold-start drafting + continual refining |
| 对本项目价值 | 工程闭环和验证体系 | 进化算子与 lineage 设计 | CANN/NPU 经验库和 RL 检索设计 |

## 对本项目的初步技术路线启发

1. 建立 CANN 版 verifier：反作弊/接口约束、编译、正确性、性能 profiling 四门。
2. 建立两阶段 Agent：先生成可行 Ascend C 算子，再围绕 latency/带宽/AI Core 利用率优化。
3. 建立 value-aware 经验库：每条经验按阶段维护价值，而不是只做静态标签。
4. 建立 lineage：每个算子任务保存 committed versions、失败尝试、profiling、reward。
5. 建立 playbook：把 CANN 数据搬运、tiling、pipeline、vectorization、bank conflict 等优化经验写成 Agent 可读策略。
6. 建立消融实验：无记忆、相似度检索、价值检索、跨任务记忆迁移分别比较。

