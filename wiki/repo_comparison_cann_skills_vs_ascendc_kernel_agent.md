# CANN 开源 Skill/Agent 仓库对比

调研对象：

- `https://gitcode.com/cann/skills`
- `https://github.com/lixuanfu66/AscendC-Kernel-Agent`

## 总体结论

如果目标是本项目的研究方向：CANN 算子编程 Agent、自动编译执行、性能测试、进化优化、经验库和训练轨迹沉淀，建议以 `AscendC-Kernel-Agent` 为主干开展工作，同时把 `cann/skills` 作为上游知识库、skill 库和工程规范来源。

简化判断：

- 主系统骨架：选 `AscendC-Kernel-Agent`。
- 权威知识与官方 skill：用 `cann/skills`。
- 最佳路线：`AscendC-Kernel-Agent` fork + 引入/同步 `cann/skills` 中的关键 skills。

## cann/skills 分析

### 定位

`cann/skills` 是 CANNBot 的 skill/plugin 集合，覆盖 Ascend C、PyPTO、TileLang 算子开发和 NPU 模型推理优化。

### 优点

- 覆盖面广：包含 Ascend C API 最佳实践、文档检索、环境检查、精度调试、运行时调试、性能 profiling、tiling 设计、UT/ST、代码审查等。
- 工程规范强：提供 plugin 安装、team、agent、hooks、测试框架和结构校验。
- 更接近官方/社区长期维护形态，有 changelog、tests、plugin marketplace。
- 知识资料丰富，适合作为 Agent 的 domain knowledge layer。
- `ops-profiling`、`ascendc-docs-search`、`ascendc-precision-debug`、`ascendc-runtime-debug`、`ops-precision-standard` 对我们的 verifier 和经验库非常有价值。

### 局限

- 更像“能力组件库”，不是完整的自主进化系统。
- 主要流程是阶段式开发、审查和验收，不直接提供 lineage、score-driven evolution、best/attempts、持续优化循环。
- 没有明显的 value-aware memory、Q-value、reward 更新机制。
- 对研究“经验如何反哺下一轮搜索”需要额外设计。

### 适合作为

- 上游 skill 库。
- CANN 知识库来源。
- 编译/精度/性能/调试能力模块。
- 官方流程与规范基线。

## AscendC-Kernel-Agent 分析

### 定位

`AscendC-Kernel-Agent` 是面向 Ascend C 自定义算子的自主生成与进化优化原型，直接借鉴 AVO：`Vary(P_t) = Agent(P_t, K, f)`。

### 优点

- 与本项目目标高度一致：包含 Agent Team、知识库、评分函数、evolution state、lineage、best/attempts 工作区。
- 有完整评分脚本：`scoring/score.sh` 编排 preflight、compile、deploy、pybind、correctness、performance 和 score aggregation。
- 支持 PyTorch 框架侧验证：Model vs ModelNew，使用 `torch.allclose` 判断正确性，NPU Event timing 测性能。
- 有可配置 benchmark：`scoring/configs/gelu_custom.json` 包含 seed、boundary、smoke、representative、stress 分层测试。
- Agent 角色更适合自主研究：Architect、Developer、Reviewer、Tester、Supervisor、Reporter。
- 有进化参数：`max_versions`、`stall_threshold`、`min_improvement_ratio`、`max_failed_attempts` 等。
- 工作区模型接近我们想要的轨迹数据结构：`best/`、`attempts/step_N/`、`evolution/scores/vN.json`、logs、report。

### 局限

- 更像研究原型，工程成熟度和长期维护性不如 `cann/skills`。
- 依赖真实 Ascend 环境：CANN、torch_npu、PyTorch、NPU、msopgen/custom operator 工具链。
- 当前重点是 AVO 式 evolution，还没有实现 EvoKernel 那种 value-driven memory/Q-value 检索。
- 仓库中 `.claude/skills` 看起来包含 `cann/skills` 的一部分拷贝，后续需要避免知识库分叉过旧。
- 评分脚本和 agent 协议需要先在本地 CANN 环境跑通，才能作为稳定实验平台。

### 适合作为

- 本项目主干原型。
- benchmark 和 scoring pipeline 基础。
- lineage/evolution 实验框架。
- 后续扩展经验库、value-aware retrieval、训练数据采集的主体。

## 对比表

| 维度 | cann/skills | AscendC-Kernel-Agent | 判断 |
| --- | --- | --- | --- |
| 定位 | Skill/plugin 能力库 | 自主算子 Agent 原型 | 主干选后者 |
| CANN 知识覆盖 | 很强 | 中等，且可能拷贝自前者 | 知识用前者 |
| 编译执行闭环 | 分散在多个 skill 中 | `scoring/score.sh` 已集成 | 后者更直接 |
| 正确性测试 | 有 precision/std/debug 能力 | 有分层 correctness pipeline | 后者更适合实验 |
| 性能测试 | `ops-profiling` 很完整 | NPU Event + score 聚合 | 两者结合 |
| Agent team | 有 team/plugin | 有面向进化的 6-Agent team | 后者贴近研究 |
| lineage | 不突出 | 明确 best/attempts/scores/state | 后者明显胜出 |
| 经验库 | 有知识组件，无动态价值更新 | 有轨迹雏形，无 Q-value memory | 需我们补齐 |
| 工程成熟度 | 更成熟 | 原型化 | 前者更稳 |
| 研究可改造性 | 需要搭主系统 | 已有主系统 | 后者更省力 |

## 推荐技术路线

### 1. 以 AscendC-Kernel-Agent 为主干

理由：

- 它已经具备我们研究所需的核心结构：`P_t` lineage、`K` 知识库、`f` 评分函数。
- 它的 scoring pipeline 可以直接承担 verifier 的角色。
- 它的工作区和分数 JSON 可以扩展为经验库和训练轨迹数据。

### 2. 把 cann/skills 作为上游知识与工具层

优先引入：

- `ascendc-docs-search`
- `ascendc-api-best-practices`
- `ascendc-npu-arch`
- `ascendc-tiling-design`
- `ascendc-precision-debug`
- `ascendc-runtime-debug`
- `ops-profiling`
- `ops-precision-standard`
- `ascendc-code-review`
- `ascendc-env-check`

### 3. 我们需要补的关键模块

- Value-aware experience memory：为经验条目维护 `Q_compile`、`Q_correctness`、`Q_performance`、`Q_reuse`。
- 经验入库器：从 `score.json`、compile logs、profiling、DESIGN/PLAN/REVIEW 自动蒸馏经验。
- 检索器：在 Agent planning 时按算子类型、失败类型、性能瓶颈和 Q-value 检索经验。
- 数据集导出器：把每轮 `attempt` 转成 SFT/DPO/过程奖励模型样本。
- CANN benchmark 管理：规范 spec、config、reference、target metric。

## 最终建议

基于 `AscendC-Kernel-Agent` 开展工作最合适，但不要直接丢掉 `cann/skills`。正确姿势是：

```text
AscendC-Kernel-Agent = 主循环、评分、进化、实验框架
cann/skills = 官方技能、知识库、调试/性能/精度工具来源
本项目新增 = value-driven memory、经验库、训练语料、RL 闭环
```

