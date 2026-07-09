## 第5天：张量并行（Tensor Parallelism）深入

### 1) 题目与考察核心
**题目**：设计并实现一个基于Megatron-LM的张量并行（TP）推理/训练引擎，支持70B模型在8卡节点内的切分。
**考察核心**：理解TP中的列切分（Column-wise）、行切分（Row-wise）及Attention层的TP实现。

### 2) 需求澄清与指标定义
- **TP Degree**：8路张量并行（TP=8），适用于单节点8卡（如HGX H100）。
- **通信开销**：TP内部的All-Reduce通信延迟需控制在微秒级，带宽利用率 > 80%。

### 3) 核心架构/技术组件设计
- **Megatron-LM TP Core**：实现Linear层的列切分（Q/K/V投影）和行切分（Output投影）。
- **All-Reduce通信池**：使用NCCL库管理TP间的通信流。

### 4) 关键技术深入与可能解
- *Linear层切分*：
  - 列切分（Column-Parallel Linear）：输入相同，权重按列切分，输出需All-Reduce。
  - 行切分（Row-Parallel Linear）：权重按行切分，输入需广播（Broadcast）或复制，输出无需All-Reduce。
- *Attention TP*：Q/K/V投影采用列切分，Attention输出采用行切分。

### 5) Trade-off（权衡）分析
- *TP粒度*：TP=8通信频繁但单卡显存低；TP=1显存高但无法运行大模型。TP度越大，All-Reduce通信开销呈线性增加。

### 6) 如何确定最优解
- 通常TP度等于节点内GPU数量（如8或4），以利用NVLink全互联拓扑，最小化跨节点通信。

### 7) 名词和缩写全称及解释
- **All-Reduce**：分布式通信原语，所有GPU贡献数据，求和后分发到所有GPU。
- **NCCL (NVIDIA Collective Communications Library)**：NVIDIA提供的分布式通信库，支持All-Reduce, All-Gather等。
- **Column-Parallel / Row-Parallel Linear**：张量并行中线性层的两种切分方式。

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
