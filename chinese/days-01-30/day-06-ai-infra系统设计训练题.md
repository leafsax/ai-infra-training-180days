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

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 推理服务）**：
  ```mermaid
  graph TD
      A[客户端/用户] --> B[API网关 / 负载均衡器]
      B --> C[vLLM/TGI 推理引擎]
      C --> D[GPU集群 H100/A100]
      C --> E[模型权重存储 NVMe/S3]
      C --> F[KV Cache 管理器]
      D --> G[GPU监控 DCGM]
  ```

- **数据流图（Data Flow Diagram - 推理流程）**：
  ```mermaid
  flowchart LR
      A[用户提示词] --> B[API网关]
      B --> C[请求队列与批处理]
      C --> D[GPU推理计算]
      D --> E[Token生成]
      E --> F[响应流式输出]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于推理批处理与延迟**：你提到了静态批处理与连续批处理。在中途动态拆分批处理时，内核融合（kernel fusion）的确切开销是多少？当长上下文请求（如128K token）到达并迫使为较短请求进行KV cache驱逐时，如何保证TP99延迟？
- **关于分布式训练通信**：在DP/TP/PP中，如何在1024个GPU的集群中处理straggler（慢节点）节点？如果一个GPU慢5%，整个All-Reduce步骤会阻塞吗？对于使用BF16权重和梯度的NCCL All-Reduce，每步的确切通信量是多少？
- **关于内存优化**：你提到了ZeRO和PagedAttention。对于ZeRO-3，如何在优化器步骤期间最小化跨节点通信开销？对于PagedAttention，当HBM发生碎片化时，确切的驱逐策略是什么（LRU vs LFU vs 基于注意力分数）？
