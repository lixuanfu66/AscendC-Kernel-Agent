# CannBot 共享知识索引

本文件记录 CannBot 当前会引用的本地共享知识库入口。共享知识库位置：

```text
D:\code\_workspace\wiki
```

## 推荐共享目录

```text
D:\code\_workspace\wiki\
  10-projects\
    CannBot.md
  20-areas\
    cann-ascendc-agent.md
    kernel-agent-memory-evolution.md
  30-resources\
    papers\
    cann\
    commands\
    templates\
  40-decisions\
    cannbot-baseline-mainline.md
```

## CannBot 优先引用的共享知识

| 类型 | 共享位置建议 | 用途 |
| --- | --- | --- |
| 项目总览 | `10-projects\CannBot.md` | 从跨项目视角记录 CannBot 的目标、当前阶段和稳定产物 |
| CANN Agent 方法 | `20-areas\cann-ascendc-agent.md` | 记录 Ascend C 算子 Agent 的通用架构、流程和经验 |
| Kernel Agent memory/evolution | `20-areas\kernel-agent-memory-evolution.md` | 跟踪 memory、自进化、verifier、strategy pool 等通用方法 |
| 论文卡片 | `30-resources\papers\*.md` | 沉淀 AutoKernel、AVO、EvoKernel、CUDA Agent、KernelSkill、cuPilot 等论文 |
| CANN 通用经验 | `30-resources\cann\*.md` | 编译、运行、精度、profiling、tiling、常见错误 |
| 命令与模板 | `30-resources\commands\*.md`、`30-resources\templates\*.md` | 保存多项目可复用的命令和经验条目模板 |
| 长期决策 | `40-decisions\*.md` | 保存跨项目稳定决策，不替代 CannBot 内的短周期计划 |

## 检索优先级

1. 先检索 `D:\code\CannBot`：当前代码、baseline、任务计划、实验状态。
2. 再检索 `D:\code\_workspace\wiki`：通用论文、方法、CANN 经验、跨项目决策。
3. 如果共享知识与项目内文档冲突，以当前项目代码和 `task_plan.md`、`progress.md`、`findings.md` 为准，并把冲突记录到 `findings.md`。

## 同步规则

- 从 CannBot 同步出去：只同步稳定结论、通用经验和可复用模板。
- 从 `_workspace` 引用回来：只登记路径、摘要和适用场景，不全文复制。
- 远程昇腾机器只作为调试环境；远程环境信息不作为本地共享知识库的主来源。
