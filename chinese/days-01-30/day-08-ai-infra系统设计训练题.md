## 第8天：LLM推理服务系统架构

### 1) 题目与考察核心
**题目**：设计一个企业级LLM Serving系统，支持多模型、多租户、高可用。
**考察核心**：整体架构设计，包括Router、Engine、GPU集群管理。

### 2) 需求澄清与指标定义
- **并发请求**：5000 QPS。
- **多模型支持**：同时部署Llama-3-70B, Qwen-14B, Embedding模型。
- **指标**：系统可用性 99.99%，路由延迟 < 10ms。

### 3) 核心架构/技术组件设计
- **LLM Router**：负责请求分发、负载均衡、模型选择。
- **Serving Engine**：如vLLM, TGI, TensorRT-LLM。
- **Model Registry**：模型版本管理与元数据管理。

### 4) 关键技术深入与可能解
- *路由策略*：基于轮询（Round-Robin）、最少连接（Least Connections）、或基于模型容量（Capacity-based）的路由。

### 5) Trade-off（权衡）分析
- *集中式Router vs 分布式Router*：集中式Router简单但可能成为瓶颈；分布式Router（如Sidecar模式）扩展性好但增加网络跳数。

### 6) 如何确定最优解
- 采用Kubernetes + Istio进行服务网格路由，结合自定义LLM Router（如vLLM's OpenAI-compatible API server）处理模型特定逻辑。

### 7) 名词和缩写全称及解释
- **LLM Serving**：大语言模型推理服务，将训练好的模型部署为API供外部调用。
- **Router (路由)**：请求分发组件，根据策略将请求导向合适的引擎或模型。

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
