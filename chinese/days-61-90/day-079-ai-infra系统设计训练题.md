### Day 79: RAG 评估与指标 (RAG Evaluation and Metrics)

**1) 题目与考察核心**：
设计 RAG 系统评估框架，考察检索指标与生成指标。

**2) 需求澄清与指标定义**：
- 检索指标：Hit Rate, MRR (Mean Reciprocal Rank)。
- 生成指标：Faithfulness（忠实度），Answer Relevance。

**3) 核心架构/技术组件设计**：
- 评估管道：使用 RAGAS 或 LlamaIndex Eval。

**4) 关键技术深入与可能解**：
- 人工评估 vs LLM-as-a-Judge：LLM 评估成本低但可能有偏差。

**5) Trade-off（权衡）分析**：
- 评估成本 vs 评估准确性。

**6) 如何确定最优解**：
采用 LLM-as-a-Judge 进行大规模评估，抽样人工验证。

**7) 名词和缩写全称及解释**：
- **MRR (Mean Reciprocal Rank)**：平均倒数排名，检索评估指标。
- **RAGAS**：RAG 评估框架。

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
