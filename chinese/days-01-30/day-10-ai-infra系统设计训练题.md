## 第10天：KV Cache管理与PagedAttention

### 1) 题目与考察核心
**题目**：设计LLM推理中的KV Cache管理系统，解决显存碎片和OOM（Out Of Memory）问题。
**考察核心**：深入理解PagedAttention算法及vLLM架构。

### 2) 需求澄清与指标定义
- **KV Cache显存占比**：在长序列推理中，KV Cache可占GPU显存的70%以上。
- **指标**：KV Cache分配延迟 < 1ms，显存碎片率 < 5%。

### 3) 核心架构/技术组件设计
- **Logical vs Physical Blocks**：将KV Cache划分为固定大小的Block（如16个Token）。
- **Block Manager**：管理物理Block的分配与释放。

### 4) 关键技术深入与可能解
- *传统KV Cache*：预分配最大序列长度的连续显存，导致严重浪费和碎片。
- *PagedAttention*：借鉴操作系统虚拟内存分页技术，将KV Cache分散存储在非连续的Physical Blocks中，通过Page Table映射。

### 5) Trade-off（权衡）分析
- *PagedAttention vs 预分配*：PagedAttention增加了一次Page Table查找开销（通常在L1 Cache中，开销可忽略），但消除了90%以上的显存浪费。

### 6) 如何确定最优解
- 全面采用PagedAttention（如vLLM引擎），并配合Prefix Caching（前缀缓存）复用相同前缀的KV Blocks。

### 7) 名词和缩写全称及解释
- **KV Cache (Key-Value Cache)**：在LLM自回归生成中，缓存过去Token的Key和Value状态，避免重复计算。
- **PagedAttention**：vLLM提出的注意力算法，使用分页机制管理KV Cache显存。
- **OOM (Out Of Memory)**：内存/显存溢出，导致程序崩溃。

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
