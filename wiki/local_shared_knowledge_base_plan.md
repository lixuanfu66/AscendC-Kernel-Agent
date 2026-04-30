# 本地跨项目共享知识库设计

## 目标

在本地建立一个多个项目可复用的共享知识库，主要服务于：

- CANN / Ascend C 算子 Agent 研究。
- CUDA / NPU kernel agent 论文与开源工程跟踪。
- 通用 Agent memory、自进化、verifier、benchmark 方法论。
- 多项目之间可复用的工程经验、调研结论和模板。

远程服务器只作为运行和调试环境，不作为主要知识中心。

## 推荐位置

经检查，本地已经存在共享 wiki：

```text
D:\code\_workspace\
  README.md
  wiki\
    00-inbox\
    10-projects\
    20-areas\
    30-resources\
    40-decisions\
    90-archive\
    templates\
```

因此不建议再新建 `D:\code\_shared_knowledge`。推荐直接将 `D:\code\_workspace` 作为本地跨项目共享知识库。

各项目通过链接或引用使用：

```text
D:\code\CannBot\
  workspace_wiki -> D:\code\_workspace\wiki

D:\code\OtherProject\
  workspace_wiki -> D:\code\_workspace\wiki
```

Windows PowerShell 创建目录链接：

```powershell
New-Item -ItemType Junction -Path D:\code\CannBot\workspace_wiki -Target D:\code\_workspace\wiki
```

如果不想用 junction，也可以只在项目文档中记录共享知识库路径。

## 基于现有 _workspace 的重新设计

```text
D:\code\_workspace\
  README.md

  wiki/
    00-inbox/
      README.md

    10-projects/
      3dVAE.md
      CannBot.md

    20-areas/
      cann-ascendc-agent.md
      kernel-agent-memory-evolution.md
      3d-generation.md

    30-resources/
      papers/
        autokernel.md
        avo.md
        evokernel.md
        cuda-agent.md
        kernelskill-kernelmem.md
        cupilot.md
      cann/
        ascendc-skill-map.md
        common-errors.md
        compile-debug.md
        runtime-debug.md
        precision-debug.md
        profiling.md
        tiling-patterns.md
      commands/
        git.md
        remote-debug.md
      templates/
        experience-entry.md
        paper-card.md
        repo-card.md
        experiment-record.md

    40-decisions/
      workspace-layout.md
      wiki-language-policy.md
      document-encoding-policy.md
      cannbot-baseline-mainline.md

    90-archive/
```

## 共享什么

### 应进入共享知识库

- 论文深度总结。
- 开源项目对比。
- CANN skills 分析。
- verifier / memory / evolution / strategy pool 方法论。
- 通用 CANN 错误和修复经验。
- 通用命令和模板。
- 多项目都可能复用的经验。

### 保留在项目内

- 项目实施计划。
- 项目-specific baseline。
- 当前项目代码设计决策。
- 当前项目实验日志。
- 未整理的 attempts、logs、raw output。

## CannBot 当前文档如何拆分

建议同步到本地共享知识库：

```text
wiki/related_work_kernel_agents.md
wiki/deep_synthesis_kernel_agent_memory_evolution.md
wiki/tracking_plan_kernel_agent_memory_evolution.md
wiki/repo_comparison_cann_skills_vs_ascendc_kernel_agent.md
wiki/cross_project_shared_memory_plan.md
```

建议保留在 CannBot 项目内：

```text
wiki/baseline_integration_plan_ascendc_kernel_agent_with_cann_skills.md
wiki/agent_operator_workflow.md
wiki/knowledge_base_design.md
wiki/rl_evolution_loop.md
wiki/training_corpus_design.md
baseline/BASELINE_v0.md
task_plan.md
progress.md
findings.md
```

边界原则：

- “研究领域通用结论”同步共享。
- “CannBot 当前实现怎么做”留在项目内。

## 维护方式

### 推荐方式：沿用 `_workspace` 独立知识仓库

优点：

- 多项目引用同一知识库。
- 可单独版本管理。
- 不和某个项目代码提交混在一起。

建议：

```text
D:\code\_workspace\.git
D:\code\CannBot\.git
D:\code\OtherProject\.git
```

### 项目内引用方式

每个项目维护一个入口文件：

```text
shared_knowledge_index.md
```

内容示例：

```markdown
# Shared Knowledge Used by This Project

- D:\code\_shared_knowledge\papers\evokernel.md
- D:\code\_shared_knowledge\agents\memory-patterns.md
- D:\code\_shared_knowledge\cann\profiling.md
```

## 与 Agent memory 的关系

本地共享知识库是“人类可读、跨项目复用”的知识层。

项目内 Agent memory 是“任务执行时可检索、可更新”的运行层。

关系：

```text
D:\code\_workspace\wiki
  -> 项目 memory/long_term 的来源

项目 workspace/runs/*
  -> 项目 memory/short_term 的来源

项目内沉淀出的通用经验
  -> 人工筛选后回写 D:\code\_workspace\wiki
```

## 初始落地命令

现有 `D:\code\_workspace` 已创建，无需重建。

可选：为 CannBot 建立入口链接。

```powershell
New-Item -ItemType Junction -Path D:\code\CannBot\workspace_wiki -Target D:\code\_workspace\wiki
```

## 第一批迁移建议

第一批不要搬太多，先复制/整理 5 类：

1. kernel agent 论文综述。
2. memory/evolution/verifier 综合。
3. CANN skill/repo 对比。
4. 技术雷达 query 和跟踪计划。
5. 经验条目模板。

## 一句话原则

`D:\code\_workspace\wiki` 保存“跨项目可复用的稳定知识”，项目 wiki 保存“当前项目的计划、实现和实验过程”。
