### Day 43: 推理缓存策略：Prompt Caching与KV Cache Caching

**1) 题目与考察核心**
设计高效的推理缓存系统。考察核心：Prompt Caching与KV Cache重用的机制。

**2) 需求澄清与指标定义**
- **QPS**：3000。
- **延迟指标**：TTFT降低30%通过缓存机制。
- **上下文特征**：大量请求包含相同系统提示（System Prompt）或长文档。

**3) 核心架构/技术组件设计**
- 缓存层：Prompt Cache（存储常见system prompt或RAG文档）与KV Cache Cache（跨请求的KV块重用）。

**4) 关键技术深入与可能解**
- **Prompt Caching**：将重复的prompt部分缓存，避免每次重新计算embedding与前置层。
- **KV Cache Caching / Prefix Caching**：如vLLM的Prefix Caching，相同前缀的KV Blocks可在不同请求间共享。

**5) Trade-off（权衡）分析**
- 缓存提升TTFT与吞吐，但占用额外CPU/GPU显存，且需处理缓存失效（Cache Eviction）策略。

**6) 如何确定最优解**
启用Prefix Caching与Prompt Caching，结合LRU（Least Recently Used）缓存淘汰策略，平衡显存占用与命中率。

**7) 名词和缩写全称及解释**
- **Prefix Caching**：缓存请求的公共前缀部分（如system prompt），供后续请求重用。
- **LRU (Least Recently Used)**：最近最少使用缓存淘汰算法。

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
