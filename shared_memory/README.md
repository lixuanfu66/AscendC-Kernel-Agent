# Shared Memory Adapter

本目录只作为 CannBot 对“共享知识”的适配说明，不保存主要共享知识数据。

当前约定：

```text
本项目知识：D:\code\CannBot
跨项目共享知识：D:\code\_workspace\wiki
远程调试环境：113.46.4.206:/data/l00821447
```

## 边界

留在本项目：

- 当前代码结构、baseline、实验流程
- 当前任务计划、进度、发现
- 算子生成过程中的 attempts、logs、score、raw output
- 与 `AscendC-Kernel-Agent` 主干强绑定的实现说明

进入共享知识库：

- CANN/Ascend C 通用编译、运行、精度、性能经验
- Kernel Agent 论文和开源工程调研
- Agent memory、自进化、verifier、strategy pool 的通用方法
- 多项目可复用命令、模板、经验条目

## 检索入口

CannBot 内部统一从以下文件了解共享知识位置和检索优先级：

```text
D:\code\CannBot\wiki\shared_knowledge_index.md
```

不建议把共享 wiki 大量复制到本项目内；项目内应保留索引、摘要和与当前代码强相关的材料。
