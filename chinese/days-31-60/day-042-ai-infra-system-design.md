### Day 42: 多模型推理服务与模型集成（Ensemble）策略

**1) 题目与考察核心**
设计支持多模型共存与模型集成的推理平台。考察核心：多模型路由、Ensemble推理架构。

**2) 需求澄清与指标定义**
- **模型数**：同时支持3个不同规模的LLM（7B, 33B, 70B）。
- **QPS**：2000，需根据请求复杂度路由至对应模型。

**3) 核心架构/技术组件设计**
- 模型注册中心：管理模型版本、路由规则。
- 路由器（Router）：基于请求特征（如长度、意图）分发到对应模型服务。

**4) 关键技术深入与可能解**
- **多模型共存（Multi-Model Serving）**：不同模型部署在不同GPU节点或同一节点的不同显存分区。
- **Ensemble（集成推理）**：多个模型输出进行投票或加权融合，提高准确率但增加延迟与计算成本。

**5) Trade-off（权衡）分析**
- 多模型共存增加调度复杂度；Ensemble提升准确率但显著降低吞吐量并增加TTFT。

**6) 如何确定最优解**
采用基于路由的多模型共存，避免不必要的Ensemble；仅在关键任务（如安全审核）中使用双模型Ensemble。

**7) 名词和缩写全称及解释**
- **Ensemble**：集成学习，通过组合多个模型的预测结果提高整体性能。

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
