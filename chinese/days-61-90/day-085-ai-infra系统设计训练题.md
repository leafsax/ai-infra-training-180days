### Day 85: 模型服务自动扩缩容 (Model Serving Auto-Scaling)

**1) 题目与考察核心**：
设计 vLLM/TGI 服务的自动扩缩容，考察动态批处理（Dynamic Batching）与模型卸载（Model Unloading）。

**2) 需求澄清与指标定义**：
- 批处理策略：Continuous Batching。
- 模型卸载：空闲 > 10 分钟卸载模型到 CPU 存储。

**3) 核心架构/技术组件设计**：
- 服务引擎：vLLM 或 TGI。
- 扩缩容：基于请求队列长度与 TTFT 指标。

**4) 关键技术深入与可能解**：
- Static Batching vs Continuous Batching：Static 按批次固定大小，Continuous 动态加入新请求。

**5) Trade-off（权衡）分析**：
- Continuous Batching 提高吞吐量但增加调度复杂度。

**6) 如何确定最优解**：
采用 vLLM + Continuous Batching + PagedAttention，结合 KEDA 基于队列长度扩缩容。

**7) 名词和缩写全称及解释**：
- **Static Batching**：静态批处理，将请求按固定批次大小分组。
- **Continuous Batching**：连续批处理（也称 In-flight Batching），在生成过程中动态插入新请求。
- **PagedAttention**：vLLM 使用的内存管理技术，将 KV Cache 分页存储。

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
