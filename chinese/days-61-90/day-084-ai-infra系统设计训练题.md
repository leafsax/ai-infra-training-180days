### Day 84: GPU 感知的自动扩缩容 (GPU-Aware Auto-Scaling)

**1) 题目与考察核心**：
设计 GPU 感知的调度与扩缩容，考察 GPU 利用率、HBM 使用与节点调度。

**2) 需求澄清与指标定义**：
- GPU 指标：GPU Utilization, HBM Usage, Temperature。
- 扩缩容目标：保持 GPU 利用率在 60-80%。

**3) 核心架构/技术组件设计**：
- GPU 监控：DCGM (Data Center GPU Manager) 收集指标。
- 调度器：K8s GPU 调度器（如 device-plugin）。

**4) 关键技术深入与可能解**：
- 基于 GPU Util vs HBM Usage：HBM 满会导致 OOM，即使 GPU Util 低。

**5) Trade-off（权衡）分析**：
- 激进扩缩容 vs 资源浪费。

**6) 如何确定最优解**：
监控 GPU Util + HBM Usage + 队列长度，设置 HBM 阈值触发扩容。

**7) 名词和缩写全称及解释**：
- **DCGM (Data Center GPU Manager)**：NVIDIA GPU 监控与管理工具。
- **HBM (High Bandwidth Memory)**：高带宽内存，GPU 使用的显存类型。

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


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于RAG检索与一致性**：在你的RAG架构中，如何处理文档更新时的embedding drift（嵌入漂移）或向量数据库一致性？为了满足TTFT < 200ms的SLA，检索步骤与LLM生成步骤分配的确切延迟预算是多少？
- **关于数据管道吞吐量**：你提到了使用Spark/Flink的ETL/ELT管道。如何防止数据加载器在训练期间成为瓶颈？对于会导致某些GPU工作节点饥饿的数据分布倾斜问题，你的策略是什么？
- **关于自动扩缩容的边缘情况**：对于基于自定义指标（如KV cache利用率）的扩缩容，如果指标服务器本身不可用会发生什么？如何设计回退机制以防止无限扩缩容或pod突然终止？
