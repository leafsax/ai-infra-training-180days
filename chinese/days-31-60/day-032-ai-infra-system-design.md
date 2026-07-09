### Day 32: Batching策略：Static vs Dynamic vs Continuous Batching

**1) 题目与考察核心**
设计支持高并发LLM推理的批处理系统。考察核心：不同Batching策略的机制、实现及性能影响。

**2) 需求澄清与指标定义**
- **QPS**：5000 QPS。
- **延迟指标**：TTFT < 300ms，TP99 < 5秒。
- **吞吐量TPS**：15000 tokens/s。
- **显存HBM**：16x A100 80GB，总显存1280GB。

**3) 核心架构/技术组件设计**
- 请求队列：使用Kafka或Redis Queue缓存 incoming requests。
- Batcher组件：负责收集请求并组成batch，送入GPU推理引擎。
- GPU执行层：执行forward pass，返回tokens。

**4) 关键技术深入与可能解**
- **Static Batching**：按固定时间窗口或固定batch size分组。优点是调度简单，缺点是batch内若某请求提前结束，GPU计算资源浪费。
- **Dynamic Batching**：在满足batch size或超时阈值时触发推理。灵活性高，但可能引入调度抖动。
- **Continuous Batching**：支持token级别的动态调度，请求完成后立即释放其KV Cache，新请求可填补空隙。

**5) Trade-off（权衡）分析**
- Static Batching保证延迟上限稳定，但吞吐低；Continuous Batching吞吐最高，但需复杂的状态管理和KV Cache追踪。

**6) 如何确定最优解**
对于高并发、长文本场景，选择Continuous Batching（如vLLM的In-flight Batching），配合PagedAttention管理KV Cache，确保高吞吐与低延迟。

**7) 名词和缩写全称及解释**
- **Batching**：将多个请求合并为一个批次进行并行推理的技术。
- **KV Cache**：Key-Value Cache，用于缓存自回归模型生成过程中计算过的key和value，避免重复计算。
- **In-flight Batching**：即Continuous Batching，支持在生成过程中动态添加/移除请求。

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
