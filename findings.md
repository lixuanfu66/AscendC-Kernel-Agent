# 研究发现与知识沉淀

## 2026-04-29 初始研究框架

### 研究对象

本项目关注“昇腾 CANN 算子编程 Agent”，不是单纯写一个算子，而是研究一个可迭代进化的智能开发系统。系统需要同时管理知识、流程、工具执行、性能优化、经验沉淀和训练数据生产。

### 知识库候选类型

1. 官方文档知识：CANN 编程模型、Ascend C、TBE/TIK、算子开发指南、编译与部署说明。
2. API 与代码知识：API 参考、样例算子、模板代码、工程结构、编译脚本。
3. 硬件与性能知识：AI Core 架构、存储层级、数据搬运、tiling、并行策略、向量化、流水。
4. 诊断知识：编译错误、运行错误、精度错误、性能异常、profiling 解释。
5. 任务知识：算子规格、输入输出 shape/dtype/layout、边界条件、参考实现、测试数据。
6. 经验知识：历史调试轨迹、优化尝试、失败反例、可复用模式。

### 初步知识组织原则

- 按用途分层：基础概念、API 参考、样例模板、问题诊断、性能优化、历史经验。
- 按任务索引：算子类型、输入输出特征、数据类型、shape 模式、layout、目标硬件。
- 按问题索引：编译错误、运行错误、精度错误、性能瓶颈、资源限制。
- 按证据强度标注：官方文档优先，成功实验其次，经验推断需保留适用边界。

### Agent 流程候选阶段

1. 解析算子规格。
2. 分析计算流程和数据依赖。
3. 制定实现计划。
4. 检索相关知识和历史经验。
5. 生成或修改代码。
6. 编译并解析错误。
7. 执行正确性测试。
8. 运行性能测试和 profiling。
9. 分析瓶颈并提出优化。
10. 多轮迭代直到达到正确性和性能目标。
11. 总结经验并入库。

### 经验条目初步结构

```yaml
id:
title:
task_type:
operator_type:
hardware:
cann_version:
context:
symptom:
hypothesis:
retrieved_knowledge:
actions:
results:
final_fix:
performance_delta:
applicability:
failure_modes:
confidence:
source_trace:
created_at:
```

### 强化学习闭环初步想法

- 环境：CANN 编译执行环境、测试集、profiling 工具、知识库和经验库。
- 状态：当前代码、规格、日志、性能数据、检索结果、历史尝试。
- 动作：检索、计划、编辑代码、调整 tiling、调整数据搬运、调整并行、重跑测试、写经验。
- 奖励：编译通过、结果正确、性能提升、迭代成本降低、经验复用价值提升。
- 数据来源：真实任务轨迹、自动生成任务、历史失败修复过程、专家标注偏好。

## 待研究关键技术问题

- 如何评估某个知识库对算子编程 Agent 的边际贡献。
- 如何避免经验库积累噪声和错误泛化。
- 如何把长编译日志和 profiling 数据压缩为可训练的关键决策轨迹。
- 如何定义性能优化中的过程奖励，而不仅是最终性能指标。
- 如何让经验库参与下一轮任务，而不是只做事后总结。
- 如何构建可复现的 CANN 算子 benchmark 和评测协议。

## 已建立的项目知识结构

- `wiki/`：长期研究 wiki，保存稳定设计文档。
- `experience/`：经验库，保存推理轨迹摘要、成功经验、失败反例和适用边界。
- `datasets/`：训练语料和评测数据设计。
- `experiments/`：benchmark、实验运行和结果分析。

首批 wiki 文档：

- `wiki/knowledge_base_design.md`
- `wiki/agent_operator_workflow.md`
- `wiki/rl_evolution_loop.md`
- `wiki/training_corpus_design.md`

## 2026-04-29 相关论文调研发现

已新增 `wiki/related_work_kernel_agents.md`，总结三篇和本项目高度相关的论文：

- AutoKernel：强调模型级 profiling、Amdahl 定律排序、单文件 edit-benchmark-keep/revert 闭环、五阶段正确性验证和优化 playbook。
- AVO：把进化搜索中的 variation operator 从固定采样/生成流程升级为自治 coding agent，利用 lineage、知识库和执行反馈做连续进化。
- EvoKernel：最贴近 CANN/NPU 场景，把 Ascend C kernel synthesis 建模为 memory-based RL，使用 value-driven retrieval、阶段性 Q-value、异构经验库和多门验证器。

对本项目的直接启发：

- CANN 算子 Agent 应采用“先正确、再性能”的两阶段目标。
- 经验库需要从静态检索升级为 value-aware retrieval。
- 每次编译、正确性、性能 profiling 都应转化为 reward 和经验价值更新信号。
- 需要维护 lineage：成功版本、失败尝试、性能轨迹和关键推理摘要。
- 第一版原型可以结合 AutoKernel 的工程闭环、AVO 的进化视角和 EvoKernel 的价值驱动记忆。

## 2026-04-29 CANN skill/Agent 仓库对比

已新增 `wiki/repo_comparison_cann_skills_vs_ascendc_kernel_agent.md`。

结论：

- `cann/skills` 更适合作为上游 skill/知识库/工程规范来源，覆盖 CANN 文档检索、API 最佳实践、环境检查、精度调试、运行时调试、性能 profiling、tiling、UT/ST 和代码审查。
- `AscendC-Kernel-Agent` 更适合作为本项目主干，因为它已经具备 AVO 式 `Vary(P_t)=Agent(P_t,K,f)`、Agent Team、scoring pipeline、best/attempts lineage、evolution state 和分层测试配置。
- 推荐路线：fork/改造 `AscendC-Kernel-Agent`，同时将 `cann/skills` 作为同步的知识与工具层。

需要补齐：

- value-aware experience memory。
- 从 score/log/profiling/DESIGN/PLAN 中自动蒸馏经验。
- 经验检索时使用阶段性价值分数。
- 将 attempts/lineage 导出为训练语料。

## 2026-04-29 技术雷达策划

已新增 `wiki/tracking_plan_kernel_agent_memory_evolution.md`，用于持续跟踪 CUDA/NPU 算子编程 Agent 的 memory 与自进化机制。

已建立 `research_tracking/` 目录，包括：

- `research_tracking/README.md`
- `research_tracking/assets/query_templates.md`
- `research_tracking/weekly/2026-W18.md`

首批高优先级跟踪对象：

- CUDA Agent：agentic RL、skill-augmented CUDA development environment、CUDA-Agent-Ops-6K。
- KernelSkill / KernelMem：dual-level memory，长期 expert skills + 短期 anti-backtracking。
- EvoKernel：NPU/Ascend C value-driven memory、阶段性 Q-value。
- cuPilot：strategy 作为中间表示、roofline-guided prompting、strategy pool。
- AVO / AutoKernel：lineage-aware evolution 与工程化 edit-benchmark loop。

## 2026-04-29 Memory 与自进化深度综合

已新增 `wiki/deep_synthesis_kernel_agent_memory_evolution.md`，对 6 条线进行了统一分析：

- AutoKernel：最适合作为 CANN verifier 和 edit-benchmark 工程闭环参考。
- AVO：最适合作为 lineage evolution 和 Agentic variation 参考。
- EvoKernel：最适合作为 CANN value-aware memory 和阶段性 Q-value 参考。
- CUDA Agent：最适合作为中长期 RL 训练闭环、skill-augmented environment 和轨迹数据生产参考。
- KernelSkill/KernelMem：最适合作为立刻落地的 long/short memory 机制参考。
- cuPilot：最适合作为 strategy pool、roofline/profiling-guided prompting 和策略级进化参考。

形成的 CANN Agent 推荐组合：

```text
CANN Agent = AutoKernel verifier
           + AVO lineage evolution
           + EvoKernel value memory
           + KernelSkill long/short memory
           + cuPilot strategy pool
           + CUDA Agent data/RL pipeline
```

下一步应优先实现 5 个模块：CANN Verifier、Long/Short Memory、Value-aware Memory、Strategy Pool、Lineage Manager。

## 2026-04-29 基线代码与 cann/skills 引入方案

已新增：

- `wiki/baseline_integration_plan_ascendc_kernel_agent_with_cann_skills.md`
- `baseline/README.md`
- `baseline/skill_import_manifest.yaml`

修正后的结论：

- 基线主干采用 `AscendC-Kernel-Agent`。
- `cann/skills` 不全量进入默认上下文，而是作为上游 skill 与知识库来源。
- 经复核，第一版需要的 10 个核心 skill 已经存在于 `AscendC-Kernel-Agent/.claude/skills/`，不需要重复引入：
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
- 保留 `AscendC-Kernel-Agent` 已有的 `ascendc-msopgen-workflow`，因为它包含 custom operator/msopgen 实战坑点。
- 当前任务从“引入 10 个核心 skill”修正为“对已内置 skill 做上游差异审计，并只补充主干缺失的可选 skill”。

第一版 baseline 的目标是：custom operator pipeline + 核心 skill + verifier summary + short/long memory skeleton + strategy id 绑定。

## 2026-04-29 Baseline v0

已新增 `baseline/BASELINE_v0.md`，作为后续实现的基线定义。

Baseline v0 明确：

- 主干使用当前项目根目录，已由 `AscendC-Kernel-Agent` 提升而来。
- `external/cann-skills` 作为上游版本和差异审计来源。
- 10 个核心 skill 已内置，无需重复引入。
- 当前工作重点转为 verifier summary、long/short memory skeleton、strategy pool、lineage summary 和 dataset export。
- 最低验收标准：能运行一次 `scoring/score.sh`，即使失败也能产生结构化 score/log，并被 lineage 消费。

## 2026-04-30 跨项目共享信息层

已新增：

- `wiki/cross_project_shared_memory_plan.md`
- `shared_memory/README.md`

结论：

- 在远程机器 `/data/l00821447` 下建议建立 `_agent_shared/`，用于跨 `CannBot`、`3dVAE` 等项目共享通用信息。
- 共享内容包括机器/Docker/NPU/CANN 环境、通用 CANN 调试经验、论文调研、常用命令和经验模板。
- 不共享项目源码、私有数据、大型 raw logs、模型权重、token 或 SSH key。
- `CannBot` 可通过软链接 `shared -> ../_agent_shared` 使用共享层，并将其中通用经验导入 long-term memory。

## 2026-04-30 本地共享知识库

用户明确主要关注本地知识共享，远程服务器只是调试环境。已新增：

- `wiki/local_shared_knowledge_base_plan.md`

结论：

- 建议在本地建立 `D:\code\_shared_knowledge` 作为跨项目共享知识库。
- 各项目通过 junction 或文档引用使用共享知识库。
- 共享库保存论文调研、CANN 通用经验、Agent memory/evolution/verifier 方法论、命令和模板。
- 项目内 wiki 保留项目计划、实现细节、实验日志和 baseline。
- 远程 `_agent_shared` 降级为运行环境记录，不作为主要知识中心。

### 共享知识库位置修正

已检查本地现有 `D:\code\_workspace`：

- 已有 `wiki/00-inbox`
- 已有 `wiki/10-projects`
- 已有 `wiki/20-areas`
- 已有 `wiki/30-resources`
- 已有 `wiki/40-decisions`
- 已有 `wiki/90-archive`
- 已有 `wiki/templates`

该结构已经适合作为统一共享知识库，因此不建议另建 `D:\code\_shared_knowledge`。后续应将 CannBot 的跨项目稳定知识合并到 `D:\code\_workspace\wiki`，项目内继续保留执行计划和实现细节。
## 2026-04-30 项目内知识与跨项目知识边界

CannBot 的文档不应整体迁出到共享 wiki，否则会削弱当前项目内 Agent 的默认检索效率。

当前边界：

- `D:\code\CannBot` 保存项目内知识：当前代码结构、baseline、任务计划、进度、发现、实验流程、attempts/logs/score、与主干实现强绑定的设计。
- `D:\code\_workspace\wiki` 保存跨项目知识：论文总结、CANN/Ascend C 通用经验、Kernel Agent memory/evolution/verifier 方法、可复用命令和模板。
- `D:\code\CannBot\wiki\shared_knowledge_index.md` 作为 CannBot 引用共享知识库的统一索引。

检索策略应采用“两级检索”：

1. 先检索 CannBot 项目内文档和代码，获得当前实现上下文。
2. 再检索 `_workspace` 共享知识库，补充通用方法和稳定经验。

如果共享知识与项目内文档冲突，以当前项目代码、`task_plan.md`、`progress.md`、`findings.md` 为准，并记录冲突来源。
## 2026-04-30 GELU 远端流程测试发现

### 1. 当前机器 runtime socVersion 是 `ascend910_93`

现象：

- 初始 `msopgen -c ai_core-Ascend910B` 生成产物可编译、可部署、pybind 可构建。
- 但运行时报错：
  `binary_info_config.json of socVersion [ascend910_93] does not support opType [GeluCustom]`

原因：

- `msopgen` 默认生成的 `CMakePresets.json` 和 `op_host/*.cpp` 使用 `ascend910`。
- 按项目 skill 改成 `ascend910b` 后，包内生成 `kernel/config/ascend910b`，但当前 runtime 仍按 `ascend910_93` 查找。
- 本机 NPU `npu-smi info -t board` 显示芯片名 `9382`，CANN 平台配置中存在 `Ascend910_9382.ini`，运行时归入 `ascend910_93`。

有效修复：

```bash
sed -i s/ascend910/ascend910_93/g CMakePresets.json op_host/gelu_custom.cpp
```

后续应固化：

- scoring 或 bootstrap 阶段自动探测当前 SoC。
- 对 910B/9382 机器，优先生成 `ascend910_93`，而不是只写死 `ascend910b`。

### 2. 空 kernel 也能完整编译部署，但正确性会输出全 0

`msopgen` 生成的默认 kernel 是：

```cpp
extern "C" __global__ __aicore__ void gelu_custom(...) {
    GET_TILING_DATA(tiling_data, tiling);
    // TODO: user kernel impl
}
```

该工程可以通过 compile/deploy/pybind，但 seed correctness 中输出全 0，失败信息为 value mismatch。

这说明 verifier 必须区分：

- 注册/部署失败：`aclnn` 找不到 op 或 binary info。
- 计算实现失败：op 可调用，但输出错误。

### 3. GELU seed kernel 可用的基础实现

本次通过的基础实现策略：

- 使用 `DTYPE_X` 宏同时支持 fp32/fp16 编译变体。
- `GET_TILING_DATA` 读取 `size`。
- 按 `GetBlockNum()` / `GetBlockIdx()` 做一维均分。
- GM/UB 搬运使用 `DataCopyPad`，因此 boundary 中 `[1]`、`[7]`、`[33]` 等非对齐长度也能通过。
- 计算使用 AscendC vector API：`Mul`、`Muls`、`Add`、`Adds`、`Tanh`。

该实现 correctness 全通过，但性能仍是 baseline 级别，后续优化方向包括：

- double buffer
- 更大 tile
- fp16 内部是否转 fp32 计算的精度/性能权衡
- 使用更接近硬件优化的 fast_gelu 模板或融合策略
- 根据 shape 和 dtype 选择不同 tile/core 切分策略
