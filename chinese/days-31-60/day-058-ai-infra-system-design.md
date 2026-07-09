### Day 58: 高流量推理API的限流与节流（Rate Limiting & Throttling）

**1) 题目与考察核心**
设计高并发推理API的保护机制。考察核心：限流算法、令牌桶、漏桶与拒绝策略。

**2) 需求澄清与指标定义**
- **QPS上限**：5000 QPS，超过则需限流。
- **用户配额**：VIP用户1000 QPS，普通用户100 QPS。

**3) 核心架构/技术组件设计**
- 限流层：API Gateway集成限流组件（如Redis + Lua脚本实现的令牌桶算法）。

**4) 关键技术深入与可能解**
- **令牌桶算法（Token Bucket）**：以固定速率向桶中放入令牌，请求需获取令牌才能执行。
- **漏桶算法（Leaky Bucket）**：请求进入漏桶，以固定速率流出处理，超出容量则拒绝。

**5) Trade-off（权衡）分析**
- 令牌桶允许突发流量；漏桶平滑流量但可能拒绝合法突发请求。

**6) 如何确定最优解**
采用令牌桶算法配合用户级配额（Rate Limit per User），在Gateway层实施，返回HTTP 429 Too Many Requests时客户端可重试。

**7) 名词和缩写全称及解释**
- **Rate Limiting（限流）**：限制请求速率以保护系统不被过载。
- **Throttling（节流）**：通过延迟或拒绝请求控制流量。
- **HTTP 429**：HTTP状态码，表示“请求过多”。

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
