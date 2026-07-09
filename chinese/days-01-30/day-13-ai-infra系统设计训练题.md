## 第13天：请求路由与负载均衡

### 1) 题目与考察核心
**题目**：设计跨多节点LLM Serving集群的请求路由与负载均衡系统。
**考察核心**：Load Balancing策略，Kubernetes Service, Istio, 自定义LLM Router。

### 2) 需求澄清与指标定义
- **集群规模**：50个GPU节点，部署3个不同版本的模型。
- **指标**：路由决策延迟 < 5ms，节点间负载偏差 < 10%。

### 3) 核心架构/技术组件设计
- **Global Router**：维护各Engine的Capacity（当前Batch size、排队请求数）。
- **Health Check & Culling**：定期探测Engine健康状态，剔除过载节点。

### 4) 关键技术深入与可能解
- *Round-Robin vs Least-Requests*：轮询简单但不感知负载；最少请求数（Least Connections）更均衡但需维护状态。
- *Capacity-based Routing*：根据GPU显存剩余量和计算负载进行路由。

### 5) Trade-off（权衡）分析
- *Stateless vs Stateful Router*：无状态Router（如DNS轮询）扩展性好但负载均衡粗糙；有状态Router精确但需处理Router自身的高可用。

### 6) 如何确定最优解
- 采用有状态LLM Router（如vLLM's Router or Text Generation Inference's router），结合gRPC健康检查与容量反馈。

### 7) 名词和缩写全称及解释
- **Load Balancing (负载均衡)**：将网络或计算负载分布到多个服务器或节点上。
- **Istio**：开源服务网格（Service Mesh），提供流量管理、安全和可观测性。

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
