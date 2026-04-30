# 项目进度日志

## 2026-04-29

### 本轮输入

用户要求使用 `$personal-wiki-workflow` 和 `Planning with Files Zh` 继续项目。项目主题是研究昇腾 CANN 算子编程 Agent，重点包括知识库、编程流程、经验库/推理轨迹、强化学习闭环和增量训练语料。

### 已完成

- 检查可用技能列表，确认 `planning-with-files-zh` 可用。
- 读取 `planning-with-files-zh` 技能说明，虽然显示存在编码问题，但核心流程可识别。
- 确认 `$personal-wiki-workflow` 未出现在当前可用技能列表中。
- 检查项目根目录，未发现既有 `task_plan.md`、`findings.md`、`progress.md`。
- 创建三份规划文件：
  - `task_plan.md`
  - `findings.md`
  - `progress.md`
- 创建个人 wiki/知识库骨架：
  - `wiki/README.md`
  - `wiki/knowledge_base_design.md`
  - `wiki/agent_operator_workflow.md`
  - `wiki/rl_evolution_loop.md`
  - `wiki/training_corpus_design.md`
- 创建经验库与实验数据目录骨架：
  - `experience/README.md`
  - `experience/templates/experience_entry.md`
  - `datasets/README.md`
  - `experiments/README.md`
- 完成第一轮相关论文调研，新增：
  - `wiki/related_work_kernel_agents.md`
- 调研论文包括：
  - AutoKernel: Autonomous GPU Kernel Optimization via Iterative Agent-Driven Search
  - AVO: Agentic Variation Operators for Autonomous Evolutionary Search
  - Towards Cold-Start Drafting and Continual Refining: A Value-Driven Memory Approach with Application to NPU Kernel Synthesis
- 克隆并分析两个 CANN 相关开源仓库：
  - `external/cann-skills`
  - `AscendC-Kernel-Agent`（已提升为当前项目根目录主干）
- 新增仓库对比文档：
  - `wiki/repo_comparison_cann_skills_vs_ascendc_kernel_agent.md`
- 新增持续技术雷达策划：
  - `wiki/tracking_plan_kernel_agent_memory_evolution.md`
  - `research_tracking/README.md`
  - `research_tracking/assets/query_templates.md`
  - `research_tracking/weekly/2026-W18.md`
- 完成 6 条关键论文/工程线的深度综合，新增：
  - `wiki/deep_synthesis_kernel_agent_memory_evolution.md`
- 完成基线代码与 `cann/skills` 引入方案，新增：
  - `wiki/baseline_integration_plan_ascendc_kernel_agent_with_cann_skills.md`
  - `baseline/README.md`
  - `baseline/skill_import_manifest.yaml`
- 复核 `AscendC-Kernel-Agent/.claude/skills`，确认 10 个核心 `cann/skills` 已经内置；已修正 baseline 文档和 manifest。
- 整理 Baseline v0，新增：
  - `baseline/BASELINE_v0.md`
- 已将 `AscendC-Kernel-Agent` 提升为当前项目根目录主干代码，后续修改直接发生在主干中。

### 环境备注

- 当前执行目录映射到了沙盒路径。
- `rg --files` 执行失败，提示拒绝访问。
- 当前目录未检测到 git 仓库。

### 下一步建议

1. 制作 skill 差异审计脚本，对比 `AscendC-Kernel-Agent/.claude/skills` 与 `cann/skills/ops`。
2. 判断是否补充主干缺失的 `torch-ascendc-op-extension`、`ascendc-docs-gen`、`ascendc-performance-best-practices`、`ascendc-regbase-best-practice`。
3. 在 `AscendC-Kernel-Agent` 主干上增加 verifier summary 输出设计。
4. 实现 long/short memory skeleton 与 strategy pool 初版。

## 2026-04-30

### 本轮输入

用户希望多个项目之间共享公用信息，避免 `CannBot`、`3dVAE` 等项目的信息割裂。

### 已完成

- 设计跨项目共享信息层 `_agent_shared/`。
- 明确共享内容和隔离边界。
- 新增文档：
  - `wiki/cross_project_shared_memory_plan.md`
  - `shared_memory/README.md`

### 下一步建议

1. 在远程服务器 `/data/l00821447` 创建 `_agent_shared/` 目录。
2. 在 `CannBot` 和 `3dVAE` 中建立 `shared -> ../_agent_shared` 软链接。
3. 首批写入远程连接、Docker、CANN 环境和 NPU 使用记录。
4. 后续将共享经验导入 `CannBot/memory/long_term/`。

### 方向修正

用户指出主要关注本地知识共享，远程只是调试环境。因此新增本地共享知识库方案：

- `wiki/local_shared_knowledge_base_plan.md`

新的优先级：

1. 在本地建立 `D:\code\_shared_knowledge`。
2. 将跨项目通用 wiki、论文调研、CANN 经验和模板放入本地共享库。
3. 项目内只保留项目计划、baseline、实现决策和实验日志。
4. 远程只保存必要的运行环境记录。

### 共享库复核

检查了本地 `D:\code\_workspace`，确认它已经是现有共享 wiki：

- `wiki/00-inbox`
- `wiki/10-projects`
- `wiki/20-areas`
- `wiki/30-resources`
- `wiki/40-decisions`
- `wiki/90-archive`
- `wiki/templates`

结论：不再推荐新建 `D:\code\_shared_knowledge`，而是将 `D:\code\_workspace` 作为统一共享知识库，并在其中新增 CannBot 项目页、CANN Agent area 和 kernel-agent 相关 resources。
## 2026-04-30 项目知识边界落地

### 已完成

- 重写 `wiki/README.md`，将其定位为 CannBot 项目内高频上下文入口。
- 新增 `wiki/shared_knowledge_index.md`，登记 `D:\code\_workspace\wiki` 作为跨项目共享知识库入口。
- 重写 `shared_memory/README.md`，明确它只是共享知识适配说明，不保存主要共享知识数据。
- 明确检索优先级：先查 `D:\code\CannBot` 的当前代码、baseline、计划和实验状态，再查 `D:\code\_workspace\wiki` 的通用论文、方法和 CANN 经验。

### 结论

不整体搬移 CannBot 的 `.md` 文档。项目内保留工作记忆、工程上下文和共享知识索引；跨项目共享库只沉淀稳定、可复用、与单个仓库实现弱绑定的知识。
