# 经验库

本目录保存 CANN 算子编程 Agent 在任务过程中形成的经验、反例和推理轨迹摘要。

经验不是简单结论，而是“场景、症状、假设、行动、结果、适用边界”的组合。

## 目录建议

```text
experience/
  README.md
  templates/
    experience_entry.md
  compile/
  runtime/
  accuracy/
  performance/
  tiling/
  data_movement/
```

## 入库标准

- 有明确任务场景。
- 有可追踪证据：日志、profiling、测试结果或代码 diff。
- 记录失败尝试，尤其是容易重复犯的错误。
- 标注适用边界，避免过度泛化。
- 标注置信度，后续实验可更新。

