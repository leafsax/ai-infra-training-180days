## 第7天：序列并行与上下文并行（Sequence/Context Parallelism）

### 1) 题目与考察核心
**题目**：设计支持超长上下文（如128K或256K Token）的LLM推理/训练系统。
**考察核心**：理解Sequence Parallelism（序列并行）和Context Parallelism（上下文并行），如Ring Attention、DeepSpeed Ulysses。

### 2) 需求澄清与指标定义
- **上下文长度**：128K Tokens。
- **KV Cache显存**：128K长度下，单请求KV Cache可能占用数GB显存。
- **指标**：长上下文推理TTFT < 2s，吞吐量下降不超过30%。

### 3) 核心架构/技术组件设计
- **Ring Attention Engine**：将序列切分在TP组内，通过Ring通信传递KV状态。
- **FlashAttention集成**：使用内存高效的注意力计算内核。

### 4) 关键技术深入与可能解
- *Sequence Parallelism (SP)*：将单个序列的Token切分到多个GPU，每个GPU计算部分Token的Attention。
- *Ring Attention*：通过环形通信在GPU间传递KV Cache，避免将所有KV Cache集中在单卡。
- *DeepSpeed Ulysses*：基于序列并行的注意力优化，结合TP和SP。

### 5) Trade-off（权衡）分析
- *SP vs 增加显存*：SP引入额外通信开销（Ring通信），但允许在有限显存下处理超长序列。通信延迟可能成为瓶颈。

### 6) 如何确定最优解
- 对于>64K上下文，启用Ring Attention或FlashAttention-2 with context parallelism。根据节点内NVLink带宽评估SP的通信开销。

### 7) 名词和缩写全称及解释
- **Sequence Parallelism (SP)**：序列并行，将单个请求的序列切分到多个GPU计算。
- **Context Parallelism**：上下文并行，处理超长上下文时的并行策略。
- **Ring Attention**：通过环形拓扑在GPU间分布和通信Attention的KV状态。

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
