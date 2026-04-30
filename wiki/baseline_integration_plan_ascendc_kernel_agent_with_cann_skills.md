# 基线方案：以 AscendC-Kernel-Agent 为主干，引入 cann/skills

## 结论

基线代码建议采用：

```text
AscendC-Kernel-Agent = 主干
cann/skills          = 上游知识库与工具 skill 来源
CannBot 本项目新增   = memory / strategy / lineage / dataset 导出层
```

经复核，`AscendC-Kernel-Agent` 已经内置本文列出的 10 个核心 skill，因此第一版不需要重复复制这些 skill。更好的方式是做“分层管理”：

1. 第一层：确认主干已内置的核心 skill，建立版本/差异审计。
2. 第二层：把 `cann/skills` 作为上游，只同步有价值更新。
3. 第三层：补充主干缺失但有价值的可选 skill。
4. 第四层：把脚本型能力接入 `scoring/`、`verifier/`、`memory_extractor/`，让它们产生结构化结果。

## 基线主干

选择 `AscendC-Kernel-Agent` 的原因：

- 已有 AVO 式 `Vary(P_t)=Agent(P_t,K,f)`。
- 已有 Agent Team：Architect、Developer、Reviewer、Tester、Supervisor、Reporter。
- 已有 `scoring/score.sh`：preflight、compile、deploy、pybind、correctness、performance。
- 已有 evolution 工作区：`best/`、`attempts/`、`scores/`、`logs/`。
- 已有 benchmark config：seed、boundary、smoke、representative、stress。
- 已有部分 CANN skills 和 Knowledge-base，可作为起点。

## 引入原则

### 原则 1：先能力闭环，后知识规模

第一版基线必须先跑通：

```text
spec -> generate attempt -> compile -> correctness -> performance -> score -> update state
```

知识库先选最能提升闭环稳定性的 skill，不追求一次性覆盖全部 CANN 资料。

### 原则 2：默认上下文轻量化

只把 skill 索引、触发条件、关键规则放入默认上下文。大型 reference 只在需要时读取。

### 原则 3：工具输出结构化

凡是会影响进化决策的工具都要输出 JSON/summary：

- 环境检查。
- 编译失败摘要。
- 正确性失败摘要。
- profiling 摘要。
- code review findings。
- memory update signals。

### 原则 4：保留上游同步边界

`cann/skills` 作为上游来源，不要大改原文件。项目内新增适配层，而不是直接污染上游 skill。

## 已内置的核心 skills

以下 10 个核心 skill 已经存在于 `AscendC-Kernel-Agent/.claude/skills/`，无需重复引入。

### 1. `ascendc-docs-search`

用途：

- API 文档检索。
- 示例代码定位。
- 兼容性信息。
- online fallback。

接入位置：

- Architect：设计前必须检索相关 API/类似算子。
- Developer：使用不确定 API 前必须检索。
- Reviewer：审查 API 用法时必须检索。
- Memory extractor：把检索到的文档引用记录进经验条目。

管理方式：

- 已内置。
- 与 `cann/skills/ops/ascendc-docs-search` 做上游差异审计。
- 保留 `references/api-index.md`、`example-catalog.md`、`compatibility.md`。
- 保留 scripts：`ascend_search_client.py`、`ascend_content_fetcher.py`、`clean_markdown.py`。

### 2. `ascendc-api-best-practices`

用途：

- API 使用规则。
- DataCopy/DataCopyPad。
- Buffer/TQue/TPipe。
- repeatTimes 限制。
- Cast 和混合精度。
- API 黑名单。

接入位置：

- Developer prompt 的硬约束。
- Reviewer 静态检查规则。
- Experience long-term memory。
- Strategy pool 的 API 策略来源。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 抽取关键规则到 `memory/long_term/api_rules.yaml`。

### 3. `ascendc-tiling-design`

用途：

- 算子分类。
- 多核切分。
- UB 切分。
- Buffer 规划。
- 分支场景覆盖。

接入位置：

- Architect seed design。
- Architect optimize design。
- Developer tiling 实现。
- Reviewer 检查 tiling 合理性。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 将“算子类型 -> tiling pattern”整理成 strategy pool。

### 4. `ascendc-npu-arch`

用途：

- chip / NpuArch / SocVersion 映射。
- Ascend910B / Ascend950 差异。
- 条件编译和 arch-specific 策略。

接入位置：

- Env preflight。
- Architect 确认目标芯片。
- Developer 生成编译配置。
- Performance strategy 判断。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 将 chip metadata 转成 `configs/hardware/ascend_chips.yaml`。

### 5. `ascendc-env-check`

用途：

- 检查 NPU 是否可用。
- 检查 `ASCEND_HOME_PATH`、`ASCEND_OPP_PATH`。
- 检查 CANN 工具链。

接入位置：

- `scoring/score.sh` Step 0 preflight。
- 新增 `verifier/env_check.sh`。

管理方式：

- 已内置。
- 与上游差异审计后按需同步脚本。
- 适配输出为 JSON：`environment.json`。

### 6. `ops-precision-standard`

用途：

- 按 dtype 和算子类型确定 rtol/atol。
- 构建 correctness config。
- 处理 inf/nan、小值、随机数、量化等特殊场景。

接入位置：

- `scoring/configs/*.json` 生成。
- `test_correctness.py` 阈值选择。
- correctness failure memory 标签。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 将 dtype 阈值规则抽成 `configs/precision_thresholds.yaml`。

### 7. `ascendc-precision-debug`

用途：

- 正确性失败诊断。
- FP16/BF16/FP32 交叉定位。
- DataCopy 对齐、EnQue/DeQue、Cast、未初始化输出。

接入位置：

- Correctness failure 后触发。
- Developer repair prompt。
- Experience extractor 生成 failure memory。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 将症状-原因速查表转成 repair strategy。

### 8. `ascendc-runtime-debug`

用途：

- 运行时错误码。
- 161xxx/361xxx/561xxx。
- Kernel 查找失败、Tiling 错误、plog 解析。

接入位置：

- deploy/run failure 后触发。
- `score.sh` failure_type=runtime/deploy/pybind 时做摘要。
- Experience memory 的 runtime failure 分类。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 接入 log parser。

### 9. `ops-profiling`

用途：

- msprof 数据采集。
- CSV 指标解释。
- 瓶颈分类。
- 优化 quickref。

接入位置：

- performance passed 后进入 profiling。
- Strategy selection。
- Performance memory update。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 将 `summary.txt` 进一步解析成 `profiling_summary.json`。

### 10. `ascendc-code-review`

用途：

- 静态审查。
- API 使用、C++ 安全、编译配置、性能风险。
- 100 分制或 findings 输出。

接入位置：

- Reviewer agent。
- keep/reject 门禁之一。
- 生成 review memory。

管理方式：

- 已内置。
- 与上游差异审计后按需同步。
- 输出结构化 `review_findings.json`。

## 主干缺失但可考虑补充的 skills

经目录对照，`cann/skills/ops` 中存在、但 `AscendC-Kernel-Agent/.claude/skills` 暂未包含的候选包括：

- `torch-ascendc-op-extension`
- `ascendc-docs-gen`
- `ascendc-performance-best-practices`
- `ascendc-regbase-best-practice`
- `cann-env-setup`
- `ops-simulator`
- `torch-ops-profiler`
- PyPTO 系列

第一版只建议关注前四个，其余延后。

## 已内置但不放入第一版硬门禁的 skills

### `ascendc-direct-invoke-template`

用途：

- 直调原型。
- 快速验证 kernel 思路。

建议：

- 作为 fallback backend。
- 第一版主干仍以 `AscendC-Kernel-Agent` 的 custom operator pipeline 为主。

### `torch-ascendc-op-extension`，主干缺失，候选补充

用途：

- 将直调 kernel 接入 PyTorch。

建议：

- 用于 direct invoke backend 的 PyTorch 测试路径。
- 不替代主干的 msopgen custom operator pipeline。

### `ascendc-registry-invoke-to-direct-invoke`

用途：

- 注册调用转直调。

建议：

- 作为后续实验工具，用于加速 debug 和最小复现。

### `ascendc-st-design` / `ascendc-ut-develop` / `ascendc-whitebox-design`，已内置

用途：

- 系统测试、单测、白盒测试。

建议：

- 第二阶段引入，用于 benchmark 测试覆盖增强。

### `ascendc-docs-gen`，主干缺失，候选补充

用途：

- 生成需求分析、详细设计、README、迭代计划等文档。

建议：

- 可用于 Reporter 或最终经验报告，不进入核心闭环。

### `ascendc-performance-best-practices`，主干缺失，候选补充

用途：

- 性能优化最佳实践。

建议：

- 目前只在确认内容后引入 strategy pool。若内容和 `ops-profiling` 重叠，以 `ops-profiling` 为主。

### `ascendc-regbase-best-practice`，主干缺失，候选补充

用途：

- Ascend950 / arch35 / regbase 特性。

建议：

- 当目标芯片包含 Ascend950 时引入；910B 基线可暂缓。

## 暂不引入的 skills

以下不作为第一版 CANN kernel evolution 基线重点：

- PyPTO 系列：`pypto-*`，除非后续研究 PyPTO DSL。
- 模型推理优化系列：`model/skills/*`，目标不同。
- `ops-simulator`：可作为无板调试备选，但第一版以真实 NPU verifier 为准。
- `torch-ops-profiler`：偏 PyTorch op profiling，后续做模型级调度时再引入。

## 接入到 AscendC-Kernel-Agent 的模块设计

### 目录结构建议

```text
AscendC-Kernel-Agent/
  .claude/skills/                 # 同步选定 cann skills
  Knowledge-base/
    cann-skills/                  # 上游只读知识快照，可选
    indexes/
  verifier/
    env_check.sh
    anti_hacking_check.py
    summarize_compile_log.py
    summarize_runtime_log.py
    summarize_correctness.py
    summarize_profiling.py
  memory/
    long_term/
      api_rules.yaml
      precision_rules.yaml
      tiling_patterns.yaml
      runtime_errors.yaml
      profiling_bottlenecks.yaml
    short_term/
      README.md
    experience/
      entries/
      index.json
    strategy_pool/
      strategies.yaml
  dataset_export/
    export_attempt.py
    export_preference_pair.py
```

### scoring pipeline 改造点

当前 `scoring/score.sh` 已有：

```text
preflight -> compile -> deploy -> pybind -> correctness -> performance -> score
```

建议扩展为：

```text
preflight
  -> env_check.json
  -> anti_hacking_check.json
compile
  -> compile_summary.json
deploy / pybind / run
  -> runtime_summary.json
correctness
  -> correctness_summary.json
  -> precision_debug_hint.json
performance
  -> performance_summary.json
profiling
  -> profiling_summary.json
score
  -> score.json
memory update
  -> memory_update.json
dataset export
  -> trajectory.jsonl
```

## Agent 接入方式

### Architect

必须使用：

- `ascendc-docs-search`
- `ascendc-tiling-design`
- `ascendc-npu-arch`
- `ascendc-api-best-practices`

输出必须包含：

- 检索到的文档/样例引用。
- 算子分类。
- tiling 策略。
- strategy id。
- 风险点。

### Developer

必须使用：

- `ascendc-api-best-practices`
- `ascendc-docs-search`
- `ascendc-msopgen-workflow`，保留自 `AscendC-Kernel-Agent`

失败后按类型使用：

- compile failure：compile summary + docs search。
- correctness failure：`ascendc-precision-debug`。
- runtime failure：`ascendc-runtime-debug`。

### Reviewer

必须使用：

- `ascendc-code-review`
- `ascendc-api-best-practices`
- `ascendc-docs-search`
- `ops-precision-standard`

输出：

- `REVIEW.md`
- `review_findings.json`
- 是否允许进入 performance。

### Tester

必须使用：

- `ascendc-env-check`
- `ops-precision-standard`
- `ops-profiling`

输出：

- `score.json`
- `correctness_summary.json`
- `profiling_summary.json`

### Supervisor

新增使用：

- short-term memory。
- lineage summary。
- strategy attempts。
- failure clusters。

输出：

- redirect directive。
- blocked strategy。
- suggested strategy id。

## 最小基线版本 Milestone

### M0：仓库基线

目标：

- Fork `AscendC-Kernel-Agent`。
- 固定 upstream `cann/skills` 来源。
- 确认现有 gelu_custom scoring 能在 CANN 环境中跑通或至少 preflight 失败可结构化输出。

产物：

- 当前项目根目录作为 `AscendC-Kernel-Agent` 主干仓库。
- `external/cann-skills` 作为上游。
- `BASELINE.md`。

### M1：skill 引入

确认 10 个核心 skill 已内置：

```text
ascendc-docs-search
ascendc-api-best-practices
ascendc-tiling-design
ascendc-npu-arch
ascendc-env-check
ops-precision-standard
ascendc-precision-debug
ascendc-runtime-debug
ops-profiling
ascendc-code-review
```

继续保留主干已有：

```text
ascendc-msopgen-workflow
```

### M2：verifier 结构化输出

目标：

- 每个阶段写 JSON summary。
- `score.json` 引用这些 summary。
- failure_type 可直接驱动 repair prompt。

### M3：memory baseline

目标：

- long-term memory 从 skill/rules 抽取。
- short-term memory 从 attempts/logs 自动生成。
- experience entry 从每轮 score 自动蒸馏。

### M4：strategy pool baseline

目标：

- 建立第一批 CANN performance strategies。
- profiling bottleneck -> strategy selection。
- 每次 attempt 绑定 strategy id。

## 第一批 Strategy Pool

```yaml
- id: mte2_datacopy_granularity
  bottleneck: mte2_bound
  idea: 增大搬运粒度，避免碎片化 DataCopy，优先使用 DataCopyPad 处理非对齐。

- id: vec_reduce_cast
  bottleneck: vec_bound
  idea: 减少不必要 Cast，FP16/BF16 场景使用 FP32 中间计算但控制 cast 次数。

- id: double_buffer_pipeline
  bottleneck: no_overlap
  idea: 检查 TQue/TPipe 双缓冲、EnQue/DeQue 配对和 MTE/VEC overlap。

- id: core_balance_tiling
  bottleneck: imbalance
  idea: 调整 blockDim、尾核处理和按维度切分方式，降低核间耗时差异。

- id: ub_buffer_reuse
  bottleneck: ub_pressure
  idea: 复用中间 buffer，减少 UB 占用，避免 Alloc/Free 不配对。

- id: bank_conflict_padding
  bottleneck: bank_conflict
  idea: 调整 UB layout 或增加 padding，降低 bank conflict。
```

## 不建议的做法

- 不建议把 `cann/skills` 全量复制到 prompt 默认上下文。
- 不建议第一版同时支持 PyPTO、direct invoke、custom operator、torch extension 四条主路径。
- 不建议先做 RL 训练，应该先积累可复现 trajectory。
- 不建议跳过 correctness 直接优化性能。
- 不建议只保留成功版本，失败尝试对 memory 和训练语料同样重要。

## 最终推荐

第一版基线应是：

```text
AscendC-Kernel-Agent custom_operator pipeline
  + cann/skills 的 10 个核心 skill
  + 保留 ascendc-msopgen-workflow
  + 结构化 verifier summaries
  + long/short memory skeleton
  + strategy id 绑定
```

这会形成一个既能跑实验、又能沉淀经验、还能继续向 value-aware memory 和 RL 数据生产演进的基线。
