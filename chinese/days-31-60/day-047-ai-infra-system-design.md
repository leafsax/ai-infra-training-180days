### Day 47: 异步推理与流式响应（Streaming Responses）设计

**1) 题目与考察核心**
设计支持流式输出的LLM推理API。考察核心：异步处理、SSE（Server-Sent Events）与流式生成。

**2) 需求澄清与指标定义**
- **QPS**：2000。
- **延迟指标**：TTFT < 200ms，流式token间隔（Inter-token latency） < 50ms。

**3) 核心架构/技术组件设计**
- API层：支持SSE或gRPC流式传输。
- 推理引擎：支持逐token输出（Streaming Output）而非等待全部生成完成。

**4) 关键技术深入与可能解**
- **SSE (Server-Sent Events)**：单向HTTP流，适合推送token序列。
- **gRPC Streaming**：双向或服务端流，延迟更低但客户端实现复杂。

**5) Trade-off（权衡）分析**
- SSE兼容性好但传输效率略低；gRPC流式性能高但需客户端支持gRPC。

**6) 如何确定最优解**
对外部API采用SSE以兼容广泛客户端；内部微服务通信采用gRPC Streaming以降低延迟。

**7) 名词和缩写全称及解释**
- **SSE (Server-Sent Events)**：服务器向客户端推送事件的HTTP标准。
- **Inter-token latency**：生成连续两个token之间的时间间隔。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - GPU集群管理）**：
  ```mermaid
  graph TD
      A[集群控制器] --> B[GPU节点 H100]
      B --> C[MIG / vGPU 分区器]
      B --> D[RDMA 网络交换机]
      A --> E[自动扩缩容 KEDA]
      E --> B
  ```

- **数据流图（Data Flow Diagram - 扩缩容流程）**：
  ```mermaid
  flowchart LR
      A[流量突增] --> B[指标收集器 DCGM]
      B --> C[Scaler 控制器]
      C --> D[ Provision 新GPU Pods]
      D --> E[路由流量到新Pods]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于KV Cache与PagedAttention**：你提出了使用PagedAttention进行KV cache管理。当在不同上下文长度下HBM严重碎片化时会发生什么？在重负载下，你如何处理缓存驱逐策略，而不降低模型准确性或导致幻觉？
- **关于GPU分区（MIG/vGPU）**：对于MIG或vGPU分区，硬性隔离保证是什么？一个MIG分区中的噪音邻居是否仍会通过共享PCIe通道或CPU内存带宽影响另一个分区？如何实时测量和执行每个分区的SLO？
- **关于自动扩缩容与抖动（Thrashing）**：对于基于KEDA的自动扩缩容，启动新的vLLM pod的从零扩缩容延迟（scale-from-zero latency）是多少？当QPS在扩缩容阈值附近快速波动时（例如10秒内+20%然后-20%），如何防止扩缩容抖动？
