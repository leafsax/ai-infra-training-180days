## 第6天：流水线并行（Pipeline Parallelism）

### 1) 题目与考察核心
**题目**：设计一个支持128层LLM的流水线并行训练系统，最小化Pipeline Bubble（空泡）时间。
**考察核心**：理解GPipe、1F1B（One-Forward-One-Backward）流水线调度，以及Micro-batching技术。

### 2) 需求澄清与指标定义
- **模型层数**：128层。
- **PP Degree**：16路流水线并行（PP=16），每卡8层。
- **指标**：Pipeline Bubble < 20%的总训练时间，GPU利用率 > 85%。

### 3) 核心架构/技术组件设计
- **Micro-batch调度器**：将Global Batch Size切分为多个Micro-batches（如MB=8）。
- **1F1B调度策略**：每个GPU执行一个Micro-batch的Forward，然后立即执行Backward，保持流水线填满。

### 4) 关键技术深入与可能解
- *GPipe (Forward-then-Backward)*：简单但Bubble大。
- *1F1B (One-Forward-One-Backward)*：Google提出的调度，减少Bubble。
- *Interleaved 1F1B (Megatron PP)*：将模型层分组，多个流水线索交替执行，进一步减少Bubble。

### 5) Trade-off（权衡）分析
- *Micro-batch size*：增大MB可减少Bubble但增加单卡显存压力；减小MB降低显存但增加调度开销和Bubble。

### 6) 如何确定最优解
- 使用Interleaved 1F1B调度，并结合自动微批次调整器（如DeepSpeed的Pipeline Scheduler）动态调整MB size以匹配显存和延迟约束。

### 7) 名词和缩写全称及解释
- **Pipeline Parallelism (PP)**：流水线并行，按层切分模型。
- **Micro-batch**：微批次，全局批次（Global Batch）切分后的子批次。
- **1F1B (One-Forward-One-Backward)**：流水线调度策略，前向和反向传播交替执行以减少空闲时间。
- **Bubble (空泡)**：流水线中GPU等待上游或下游数据而产生的空闲时间。

---