# 跨项目共享信息与记忆层设计

> 更新说明：当前更推荐将主要共享知识库放在本地，例如 `D:\code\_shared_knowledge`。远程 `_agent_shared` 仅作为运行环境记录和临时调试经验，不作为主要知识中心。详见 `wiki/local_shared_knowledge_base_plan.md`。

## 背景

当前同一台昇腾机器上可能同时存在多个项目，例如：

- `CannBot`：CANN 算子编程 Agent 研究。
- `3dVAE`：已有项目，也使用同一服务器和可能的 NPU/CANN 环境。

这些项目之间存在大量可共享信息：

- 服务器、Docker、NPU、CANN 环境信息。
- 常用命令和排障经验。
- CANN 编译、运行、profiling、精度调试经验。
- 可复用论文调研和 Agent memory/evolution 方案。
- 通用 skill、工具脚本、benchmark 规范。

但项目私有信息也必须隔离：

- 项目源码。
- 项目私有数据。
- 未公开实验结果。
- 任务特定日志和模型权重。

因此需要设计一个“共享层 + 项目层”的信息结构。

## 推荐结构

在远程机器 `/data/l00821447` 下建立共享目录：

```text
/data/l00821447/
  _agent_shared/
    README.md
    machine/
      ascend_113_46_4_206.md
      docker_claude_sz_1.md
      npu_status_notes.md
      environment_checklist.md
    cann/
      cann_env.md
      common_errors.md
      compile_debug.md
      runtime_debug.md
      precision_debug.md
      profiling_notes.md
      skill_index.md
    papers/
      kernel_agent_memory_evolution.md
      paper_cards/
    experience/
      compile/
      runtime/
      correctness/
      performance/
      strategy/
    commands/
      remote_access.md
      docker.md
      npu.md
      git.md
    templates/
      experience_entry.md
      experiment_record.md
      remote_run_record.md

  CannBot/
    shared -> ../_agent_shared

  3dVAE/
    shared -> ../_agent_shared
```

项目内通过软链接访问共享层：

```bash
ln -s ../_agent_shared shared
```

这样每个项目都可以读到共享知识，但项目私有内容仍留在各自目录。

## 共享什么

### 1. 机器与环境信息

路径：

```text
_agent_shared/machine/
```

内容：

- 服务器地址：`113.46.4.206`
- SSH 端口：`101`
- Docker 容器：`claude_sz_1`
- 工程根目录：`/data/l00821447`
- NPU 使用注意事项。
- CANN toolkit 路径。
- torch/torch_npu 版本。
- 3dVAE 是否占用设备的观察记录。

示例：

```yaml
host: 113.46.4.206
ssh_port: 101
docker: claude_sz_1
workspace_root: /data/l00821447
cann_set_env: /usr/local/Ascend/ascend-toolkit/set_env.sh
shared_projects:
  - CannBot
  - 3dVAE
```

### 2. CANN 通用经验

路径：

```text
_agent_shared/cann/
_agent_shared/experience/
```

内容：

- 编译失败经验。
- msopgen 坑点。
- DataCopy/DataCopyPad 对齐问题。
- EnQue/DeQue、TPipe/TQue 同步问题。
- PyTorch/torch_npu 绑定问题。
- profiling 指标解释。
- 910B/950 架构差异。

这些经验可以被 `CannBot` 直接转成 long-term memory。

### 3. 论文和开源项目调研

路径：

```text
_agent_shared/papers/
```

内容：

- AutoKernel。
- AVO。
- EvoKernel。
- CUDA Agent。
- KernelSkill/KernelMem。
- cuPilot。
- 后续每周技术雷达。

这类内容不属于某个项目，可以跨任务复用。

### 4. 常用命令

路径：

```text
_agent_shared/commands/
```

内容：

- SSH 登录。
- Docker 进入。
- NPU 状态查看。
- CANN 环境检查。
- Git remote 管理。
- 首次部署流程。

### 5. 可复用模板

路径：

```text
_agent_shared/templates/
```

内容：

- 经验条目模板。
- 实验记录模板。
- 远程运行记录模板。
- 故障诊断模板。

## 不共享什么

以下内容默认不进入 `_agent_shared`：

- 3dVAE 项目源码。
- CannBot 未整理的原始 attempts 全量目录。
- 大型 raw logs。
- 大型数据集、模型权重。
- 用户私有 token、密码、SSH key。
- 未经过滤的敏感路径或业务数据。

如果某个项目经验有复用价值，应先蒸馏成摘要，再放入共享经验库。

## 与 CannBot memory 的关系

`_agent_shared` 是人类和多个项目共用的知识层。

`CannBot/memory/` 是 CANN 算子 Agent 的任务内 memory 层。

建议关系：

```text
_agent_shared/experience/  ->  CannBot/memory/long_term/
CannBot/workspace/runs/*   ->  CannBot/memory/short_term/
CannBot/memory/experience  ->  可筛选同步回 _agent_shared/experience/
```

也就是说：

- 共享层提供长期通用知识。
- CannBot 将共享层吸收到 long-term memory。
- CannBot 每次任务产生的经验，经筛选后再回写共享层。

## 同步策略

### 写入共享层的条件

一条信息满足任一条件即可写入：

- 未来其他项目大概率会用到。
- 是环境配置或机器状态。
- 是 CANN/CUDA/NPU 通用错误。
- 是通用性能优化经验。
- 是论文/开源项目调研。
- 是通用命令或模板。

### 不写入共享层的条件

- 只对当前项目一次性有用。
- 含敏感数据。
- 日志太长且未总结。
- 没有复现证据。

## 初始落地步骤

在远程服务器上执行：

```bash
cd /data/l00821447
mkdir -p _agent_shared/{machine,cann,papers/paper_cards,experience/{compile,runtime,correctness,performance,strategy},commands,templates}
```

为项目建立软链接：

```bash
cd /data/l00821447/CannBot
ln -s ../_agent_shared shared

cd /data/l00821447/3dVAE
ln -s ../_agent_shared shared
```

首批写入：

```text
_agent_shared/machine/ascend_113_46_4_206.md
_agent_shared/machine/docker_claude_sz_1.md
_agent_shared/commands/remote_access.md
_agent_shared/cann/cann_env.md
_agent_shared/cann/common_errors.md
_agent_shared/papers/kernel_agent_memory_evolution.md
```

## 对当前项目的建议

在 `CannBot` 中新增：

```text
shared/                        # 指向 /data/l00821447/_agent_shared
memory/long_term/shared_index.md
```

`shared_index.md` 记录当前 Agent 允许读取的共享知识入口。

初始可读入口：

- `shared/machine/`
- `shared/cann/`
- `shared/papers/`
- `shared/experience/`
- `shared/commands/`

## 后续自动化

后续可做一个同步脚本：

```text
scripts/sync_shared_memory.py
```

功能：

- 扫描 `_agent_shared/experience/`。
- 生成 `memory/long_term/shared_experience_index.json`。
- 按标签导入 CANN Agent 检索系统。
- 记录来源路径和更新时间。

## 一句话原则

共享层只放“跨项目仍然有价值的知识”，项目层保留“当前任务的完整上下文和原始产物”。
