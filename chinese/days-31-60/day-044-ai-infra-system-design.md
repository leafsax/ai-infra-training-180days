### Day 44: 推理集群的容错与高可用设计

**1) 题目与考察核心**
设计高可用的LLM推理集群。考察核心：故障检测、自动恢复与数据一致性。

**2) 需求澄清与指标定义**
- **集群规模**：50个GPU节点。
- **可用性目标**：99.9% SLA（Service Level Agreement）。

**3) 核心架构/技术组件设计**
- 健康检查（Health Check）与自动重启机制。
- 模型权重冗余存储与快速加载机制。

**4) 关键技术深入与可能解**
- **Pod Restart / Rescheduling**：K8s自动重启崩溃的推理Pod。
- **模型热备份（Hot Backup）**：模型权重存储在分布式存储，新Pod启动时快速拉取。
- **请求重试机制**：客户端或网关对失败请求进行透明重试。

**5) Trade-off（权衡）分析**
- 频繁重试可能加剧集群负载；热备份增加存储与网络开销。

**6) 如何确定最优解**
结合K8s Liveness/Readiness Probes与网关级重试（有限次数与指数退避），确保高可用同时避免雪崩。

**7) 名词和缩写全称及解释**
- **SLA (Service Level Agreement)**：服务级别协议，定义服务可用性目标。
- **Liveness/Readiness Probes**：K8s用于检测容器是否存活及是否准备好接收流量的机制。

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
