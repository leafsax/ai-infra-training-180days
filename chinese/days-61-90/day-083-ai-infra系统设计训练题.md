### Day 83: AI 服务自动扩缩容基础 (Auto-Scaling Fundamentals for AI Services)

**1) 题目与考察核心**：
设计 AI 推理服务的自动扩缩容系统，考察指标驱动扩缩容与 K8s HPA。

**2) 需求澄清与指标定义**：
- 扩缩容指标：GPU 利用率 > 70% 触发扩容，< 30% 触发缩容。
- 扩缩容延迟：< 2 分钟。

**3) 核心架构/技术组件设计**：
- 监控：Prometheus 收集 GPU 指标。
- 扩缩容控制器：K8s HPA (Horizontal Pod Autoscaler) 或 KEDA。

**4) 关键技术深入与可能解**：
- 基于请求数扩缩容 vs 基于 GPU 利用率扩缩容。

**5) Trade-off（权衡）分析**：
- 请求数扩缩容响应快但可能忽略资源瓶颈；GPU 利用率更准确但延迟高。

**6) 如何确定最优解**：
采用混合指标：QPS + GPU 利用率 + 队列长度。

**7) 名词和缩写全称及解释**：
- **HPA (Horizontal Pod Autoscaler)**：Kubernetes 水平 Pod 自动扩缩容器。
- **KEDA (Kubernetes Event-Driven Autoscaling)**：基于事件的 K8s 扩缩容工具。

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
