### Day 45: 推理服务的监控与可观测性：Metrics, Logs, Traces

**1) 题目与考察核心**
设计推理服务的可观测性系统。考察核心：Metrics（指标）、Logs（日志）、Traces（链路追踪）的采集与分析。

**2) 需求澄清与指标定义**
- **监控粒度**：节点级、Pod级、请求级。
- **指标类型**：GPU利用率、显存使用、TTFT、TPOT、QPS、错误率。

**3) 核心架构/技术组件设计**
- 指标采集：Prometheus + DCGM Exporter。
- 日志收集：Fluentd/Fluentbit -> Elasticsearch/ Loki。
- 链路追踪：OpenTelemetry / Jaeger。

**4) 关键技术深入与可能解**
- **Metrics**：时间序列数据，用于告警与HPA。
- **Traces**：记录单个请求在网关、路由、推理引擎中的完整调用链，用于定位延迟瓶颈。

**5) Trade-off（权衡）分析**
- 细粒度追踪增加计算与存储开销；需平衡监控精度与系统性能。

**6) 如何确定最优解**
采用OpenTelemetry进行请求级Tracing，Prometheus采集GPU与QPS指标，日志采用异步写入以降低推理延迟影响。

**7) 名词和缩写全称及解释**
- **Metrics**：系统运行指标数据（如CPU、GPU、延迟）。
- **Traces**：链路追踪，记录请求在分布式系统中的调用路径。
- **OpenTelemetry**：开源可观测性框架，支持指标、日志、追踪的标准化采集。

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
