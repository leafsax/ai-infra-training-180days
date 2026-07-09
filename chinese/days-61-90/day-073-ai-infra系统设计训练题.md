### Day 73: 向量数据库与索引 (Vector Databases and Indexing)

**1) 题目与考察核心**：
设计与选型向量数据库（Milvus, Pinecone, FAISS），考察索引算法与分布式架构。

**2) 需求澄清与指标定义**：
- 向量规模：10 亿向量，768 维。
- 存储需求：单向量 3KB（float32），总存储约 3 TB。
- 查询 QPS：50,000 QPS，P99 延迟 < 50 ms。

**3) 核心架构/技术组件设计**：
- 索引类型：HNSW, IVF_PQ, DiskANN。
- 分布式架构：分片（Sharding）与副本（Replication）。

**4) 关键技术深入与可能解**：
- HNSW vs IVF_PQ：HNSW 延迟低但内存占用大；IVF_PQ（乘积量化）压缩向量，节省内存但精度略降。

**5) Trade-off（权衡）分析**：
- 内存 vs 精度：HNSW 需要大量 HBM/RAM，IVF_PQ 通过量化降低精度但节省资源。

**6) 如何确定最优解**：
对于 10 亿向量，采用 IVF_PQ 或 DiskANN 以控制存储成本，对高优先级查询路由到 HNSW 索引。

**7) 名词和缩写全称及解释**：
- **HNSW**：Hierarchical Navigable Small World，层次可导航小世界图。
- **IVF_PQ (Inverted File with Product Quantization)**：倒排文件结合乘积量化，一种向量压缩与检索技术。
- **DiskANN**：基于磁盘的近似最近邻搜索框架。
- **Sharding**：数据分片，将数据分布到多个节点。

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
