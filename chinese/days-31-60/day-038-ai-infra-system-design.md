### Day 38: 推理集群的请求路由与负载均衡策略

**1) 题目与考察核心**
设计多节点推理集群的请求路由与负载均衡系统。考察核心：路由策略、状态感知负载均衡。

**2) 需求澄清与指标定义**
- **集群规模**：20个GPU节点，每节点8x GPU。
- **QPS**：5000。
- **延迟指标**：TP99 < 3秒，要求请求分布均匀，避免热点节点。

**3) 核心架构/技术组件设计**
- 路由层：API Gateway + 服务发现（Service Discovery）+ 状态感知负载均衡器（State-aware Load Balancer）。
- 监控组件：收集各节点的GPU利用率、KV Cache剩余显存、请求队列长度。

**4) 关键技术深入与可能解**
- **Round-Robin / Random Routing**：简单但无视节点状态，易导致负载不均。
- **Least Connections / Least Load**：基于活跃请求数或GPU利用率路由。
- **KV Cache-aware Routing**：将具有相似prompt的请求路由到同一节点，利用KV Cache缓存提升命中率。

**5) Trade-off（权衡）分析**
- 状态感知路由增加路由层复杂度与监控开销；简单路由易实现但吞吐与延迟表现不稳定。

**6) 如何确定最优解**
采用基于GPU利用率与请求队列长度的Least Load路由，结合KV Cache亲和性（Affinity）路由，平衡负载与缓存命中率。

**7) 名词和缩写全称及解释**
- **负载均衡（Load Balancing）**：将网络请求分发到多个服务器以提高响应速度与可靠性。
- **服务发现（Service Discovery）**：动态识别集群中可用服务实例的技术。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 高级推理服务）**：
  ```mermaid
  graph TD
      A[负载均衡器] --> B[推理 Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU资源管理器]
      D --> E[Kubernetes / Karpenter]
      B --> F[遥测 DCGM/OpenTelemetry]
  ```

- **数据流图（Data Flow Diagram - 连续批处理）**：
  ```mermaid
  flowchart LR
      A[ incoming 请求] --> B[连续批处理调度器]
      B --> C[KV Cache 查找与更新]
      C --> D[GPU Tensor Core 计算]
      D --> E[输出 Tokens]
      E --> F[流式响应]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于KV Cache与PagedAttention**：你提出了使用PagedAttention进行KV cache管理。当在不同上下文长度下HBM严重碎片化时会发生什么？在重负载下，你如何处理缓存驱逐策略，而不降低模型准确性或导致幻觉？
- **关于GPU分区（MIG/vGPU）**：对于MIG或vGPU分区，硬性隔离保证是什么？一个MIG分区中的噪音邻居是否仍会通过共享PCIe通道或CPU内存带宽影响另一个分区？如何实时测量和执行每个分区的SLO？
- **关于自动扩缩容与抖动（Thrashing）**：对于基于KEDA的自动扩缩容，启动新的vLLM pod的从零扩缩容延迟（scale-from-zero latency）是多少？当QPS在扩缩容阈值附近快速波动时（例如10秒内+20%然后-20%），如何防止扩缩容抖动？
