### Day 80: RAG 系统延迟优化 (RAG System Latency Optimization)

**1) 题目与考察核心**：
优化 RAG 系统延迟，考察缓存、异步检索与预计算。

**2) 需求澄清与指标定义**：
- 目标 TTFT < 300 ms，总延迟 < 2 秒。

**3) 核心架构/技术组件设计**：
- 查询缓存：Redis 缓存常见查询的检索结果。
- 异步检索：并行执行向量检索与重排序。

**4) 关键技术深入与可能解**：
- 缓存命中率 vs 数据新鲜度。

**5) Trade-off（权衡）分析**：
- 缓存提升延迟但增加存储与一致性问题。

**6) 如何确定最优解**：
采用 TTL 缓存（如 1 小时）+ 异步重排序，关键查询实时计算。

**7) 名词和缩写全称及解释**：
- **TTL (Time to Live)**：缓存存活时间。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 自动扩缩容）**：
  ```mermaid
  graph TD
      A[API网关] --> B[推理服务 vLLM]
      B --> C[自定义指标服务器]
      C --> D[KEDA 扩缩容器]
      D --> E[Kubernetes 集群]
      E --> B
  ```

- **数据流图（Data Flow Diagram - 扩缩容数据流）**：
  ```mermaid
  flowchart LR
      A[请求负载] --> B[指标导出器]
      B --> C[Prometheus / DCGM]
      C --> D[KEDA 操作器]
      D --> E[扩容/缩容 Pods]
  ```
