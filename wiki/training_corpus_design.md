# 训练语料设计

## 目标

把 CANN 算子编程 Agent 的任务过程转化为可训练、可评估、可审计的数据集，用于 SFT、偏好优化、过程奖励模型和检索增强评测。

## 样本结构

```yaml
sample_id:
task:
  operator_name:
  spec:
  target_hardware:
  cann_version:
context:
  retrieved_docs:
  retrieved_experiences:
trajectory:
  - step_id:
    observation:
    thought_summary:
    action:
    action_input:
    result:
    reward_hint:
artifacts:
  code_diff:
  build_log_summary:
  run_log_summary:
  profiling_summary:
outcome:
  correctness:
  performance:
  iteration_count:
  final_status:
experience:
  distilled_lesson:
  applicability:
  confidence:
quality:
  reviewer:
  score:
  issues:
```

## 数据类型

| 类型 | 用途 |
| --- | --- |
| 指令样本 | 训练 Agent 根据规格生成计划或代码 |
| 诊断样本 | 根据日志定位错误和修复方案 |
| 优化样本 | 根据 profiling 提出性能优化 |
| 偏好样本 | 比较两条轨迹优劣 |
| 过程奖励样本 | 训练每一步决策质量评估 |
| 检索评测样本 | 评估知识库是否找到了关键资料 |

## 清洗规则

- 保留关键决策，不保留冗余日志全文。
- 编译日志压缩为错误摘要、关键堆栈和代码位置。
- profiling 压缩为瓶颈指标和优化前后对比。
- 失败轨迹可以保留，但要标注失败原因和不应重复的动作。
- 经验必须标注适用边界和置信度。

## 质量评分

| 维度 | 说明 |
| --- | --- |
| 正确性 | 最终代码是否通过测试 |
| 性能 | 是否达到或接近目标 |
| 过程效率 | 迭代次数和无效尝试是否合理 |
| 依据质量 | 是否引用可靠知识或实验结果 |
| 可复用性 | 经验能否迁移到类似任务 |
| 可训练性 | 样本是否结构清晰、噪声低 |

