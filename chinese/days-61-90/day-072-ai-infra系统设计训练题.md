### Day 72: 文档解析与分块策略 (Document Parsing and Chunking Strategies)

**1) 题目与考察核心**：
设计文档解析与语义分块（Semantic Chunking）策略，优化 RAG 检索质量。

**2) 需求澄清与指标定义**：
- 文档类型：PDF, Word, Markdown, HTML。
- 分块大小：200-500 tokens/块。
- 重叠（Overlap）：50-100 tokens。

**3) 核心架构/技术组件设计**：
- 解析器：Unstructured.io 或 LlamaParse 提取文本与结构。
- 分块策略：基于规则的分块（按段落/标题）vs 语义分块（基于句子相似度）。

**4) 关键技术深入与可能解**：
- 固定大小分块 vs 语义分块：固定大小简单但可能割裂语义；语义分块保持上下文完整，但计算成本高。

**5) Trade-off（权衡）分析**：
- 分块大小：大块保留上下文但引入噪声；小块精确但可能缺乏上下文。

**6) 如何确定最优解**：
采用基于层级结构（标题、段落）的分块，结合 10-15% overlap，对核心文档使用语义分块。

**7) 名词和缩写全称及解释**：
- **Semantic Chunking**：语义分块，根据内容语义边界而非固定长度进行文本切分。
- **Overlap**：分块之间的重叠 token 数量，用于保持上下文连续性。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 数据管道）**：
  ```mermaid
  graph TD
      A[数据源] --> B[ETL/ELT 管道 Spark/Flink]
      B --> C[数据湖 S3/Parquet]
      B --> D[向量数据库 FAISS/Weaviate]
  ```

- **数据流图（Data Flow Diagram - RAG系统）**：
  ```mermaid
  flowchart LR
      A[用户查询] --> B[RAG 编排器]
      B --> C[向量数据库检索]
      C --> D[上下文构造]
      D --> E[LLM推理]
      E --> F[最终响应]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于RAG检索与一致性**：在你的RAG架构中，如何处理文档更新时的embedding drift（嵌入漂移）或向量数据库一致性？为了满足TTFT < 200ms的SLA，检索步骤与LLM生成步骤分配的确切延迟预算是多少？
- **关于数据管道吞吐量**：你提到了使用Spark/Flink的ETL/ELT管道。如何防止数据加载器在训练期间成为瓶颈？对于会导致某些GPU工作节点饥饿的数据分布倾斜问题，你的策略是什么？
- **关于自动扩缩容的边缘情况**：对于基于自定义指标（如KV cache利用率）的扩缩容，如果指标服务器本身不可用会发生什么？如何设计回退机制以防止无限扩缩容或pod突然终止？
