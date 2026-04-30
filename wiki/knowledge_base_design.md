# 知识库设计

## 目标

为 CANN 算子编程 Agent 提供可检索、可评估、可更新的知识来源，使 Agent 在规格分析、代码生成、编译诊断、性能优化和经验总结时能够获得可靠上下文。

## 知识分层

| 层级 | 内容 | 主要用途 |
| --- | --- | --- |
| L0 官方事实 | CANN 官方文档、API 参考、工具说明、版本说明 | 约束 Agent，避免编造 API 和流程 |
| L1 工程模板 | 样例算子、构建脚本、工程结构、测试模板 | 快速生成可编译工程 |
| L2 诊断知识 | 编译错误、运行错误、精度错误、profiling 解释 | 缩短 debug 回路 |
| L3 性能知识 | tiling、数据搬运、流水、并行、内存层级、向量化 | 指导优化方向 |
| L4 历史经验 | 成功/失败轨迹、优化案例、反例、适用边界 | 帮助 Agent 做类比和迁移 |
| L5 任务数据 | 算子规格、benchmark、输入分布、评价指标 | 构建实验和训练数据 |

## 元数据 schema

```yaml
doc_id:
title:
source_type: official | sample | experiment | experience | paper | note
source_url:
version:
cann_version:
hardware:
operator_types:
task_stages:
problem_types:
keywords:
confidence:
last_verified:
summary:
```

## 检索入口

按 Agent 阶段检索：

- 规格解析：算子定义、shape/dtype/layout 约束、边界条件。
- planning：类似算子样例、计算流程、tiling 模式。
- coding：API、模板、工程结构、参考实现。
- 编译：错误码、编译日志片段、版本兼容问题。
- 执行：运行失败、精度检查、测试生成。
- 性能：profiling 指标、瓶颈类型、优化策略。
- 经验总结：历史轨迹、类似失败、性能提升案例。

## 知识治理

- 每条知识要有来源和置信度。
- 官方文档优先，但实验结果可以覆盖“经验性建议”。
- 经验必须记录适用边界，避免把特定 shape、特定硬件或特定版本下的结论泛化。
- 对过期 CANN 版本或硬件的知识应保留，但检索时降权。

## 评估方法

评估某类知识库是否有帮助，可以使用消融实验：

| 实验组 | 可用知识 | 观察指标 |
| --- | --- | --- |
| baseline | 无检索或仅通用模型知识 | 编译通过率、正确率、性能 |
| official-only | 官方文档/API | API 正确性、编译错误率 |
| sample-augmented | 官方文档 + 样例 | 首次可运行率 |
| diagnosis-augmented | 加入错误案例 | 修复轮数 |
| experience-augmented | 加入历史经验 | 性能提升和迭代成本 |

