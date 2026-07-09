## 第11天：KV Cache优化（量化、驱逐、前缀缓存）

### 1) 题目与考察核心
**题目**：在多租户RAG（检索增强生成）场景下，设计KV Cache优化策略以支持高并发。
**考察核心**：KV Cache Quantization（量化）、Eviction Strategies（驱逐策略）、Prefix Caching（前缀缓存）。

### 2) 需求澄清与指标定义
- **RAG场景特征**：大量请求共享相同的System Prompt或文档前缀。
- **指标**：前缀缓存命中率 > 60%，KV Cache显存占用减少40%。

### 3) 核心架构/技术组件设计
- **Prefix Cache Manager**：基于内容哈希（Content-based Hashing）存储和复用KV Blocks。
- **LRU/KV Cache Eviction**：当显存不足时，基于最近最少使用（LRU）或Token重要性驱逐Blocks。

### 4) 关键技术深入与可能解
- *Prefix Caching*：缓存公共前缀的KV状态，新请求到来时直接挂载。
- *KV Quantization*：将KV Cache量化为INT8或INT4，减少显存占用50%-75%。

### 5) Trade-off（权衡）分析
- *Quantization vs 精度*：INT4量化可能导致生成质量轻微下降（Perplexity增加），需通过AWQ或GPTQ方法进行感知训练量化（PTQ）。

### 6) 如何确定最优解
- 结合Prefix Caching（针对RAG）和KV Cache INT8量化（针对通用生成），启用vLLM的`--enable-prefix-caching`和`--kv-cache-dtype=fp8`。

### 7) 名词和缩写全称及解释
- **Prefix Caching**：前缀缓存，复用请求间共享的初始Token的KV状态。
- **LRU (Least Recently Used)**：最近最少使用算法，用于缓存驱逐。
- **PTQ (Post-Training Quantization)**：训练后量化，在模型训练完成后进行量化，无需重新训练。

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
