# CannBot 项目 Wiki

本目录保存 **CannBot 当前项目开发所需的高频上下文**：baseline、工程集成、算子生成流程、实验流程、项目内决策和近期发现。

跨项目可复用的论文总结、CANN/Ascend C 通用经验、Agent memory/自进化方法，应沉淀到本地共享知识库：

```text
D:\code\_workspace\wiki
```

项目内只保留索引和与当前代码强绑定的材料，避免 CannBot 自己的检索入口变弱。

## 项目内核心文档

- [baseline 集成方案](baseline_integration_plan_ascendc_kernel_agent_with_cann_skills.md)
- [算子编程 Agent 工作流](agent_operator_workflow.md)
- [知识库设计](knowledge_base_design.md)
- [强化学习与进化闭环](rl_evolution_loop.md)
- [训练语料设计](training_corpus_design.md)
- [本地共享知识库设计](local_shared_knowledge_base_plan.md)
- [共享知识索引](shared_knowledge_index.md)

## 研究与调研文档

这些文档当前保留在项目内，方便 CannBot 开发阶段检索；当结论稳定后，再把可复用部分同步到 `D:\code\_workspace\wiki`。

- [Kernel Agent 相关工作](related_work_kernel_agents.md)
- [Memory/Evolution 深度综合](deep_synthesis_kernel_agent_memory_evolution.md)
- [技术跟踪计划](tracking_plan_kernel_agent_memory_evolution.md)
- [cann/skills 与 AscendC-Kernel-Agent 对比](repo_comparison_cann_skills_vs_ascendc_kernel_agent.md)

## 知识边界

留在 `D:\code\CannBot`：

- 当前项目计划、进度、发现：`task_plan.md`、`progress.md`、`findings.md`
- 当前 baseline 和代码结构说明：`baseline/`、`wiki/baseline_*.md`
- 当前算子生成、编译、执行、评分、远程调试流程
- 未清洗的实验记录、attempts、logs、raw output
- 与当前仓库实现强绑定的设计决策

沉淀到 `D:\code\_workspace\wiki`：

- 论文卡片和方法总结
- CANN/Ascend C 通用编译、运行、精度、性能经验
- CUDA/NPU Kernel Agent 的 memory、self-evolution、verifier、benchmark 方法
- 多项目可复用的命令、模板、经验条目
- 已稳定、可复用、与单个仓库代码无强绑定的技术结论

## 维护规则

- 新增文档前先判断：它服务于 CannBot 当前开发，还是可跨项目复用。
- 项目内文档可以引用共享知识库，但不要把共享知识库全文复制回来。
- 共享知识库的稳定条目，应在 [shared_knowledge_index.md](shared_knowledge_index.md) 中登记。
- 短周期任务记录留在项目内；稳定结论再人工筛选后同步到 `_workspace`。
