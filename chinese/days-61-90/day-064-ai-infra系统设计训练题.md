### Day 64: 分布式数据处理框架 (Distributed Data Processing Frameworks)

**1) 题目与考察核心**：
设计并比较用于 AI 数据处理的分布式框架（如 Apache Spark, Ray, Dask），考察框架选型、任务调度、内存管理与容错机制。

**2) 需求澄清与指标定义**：
- 数据量：PB 级非结构化与结构化数据混合处理。
- 处理延迟：批处理任务需在 4 小时内完成 10 TB 数据处理。
- 吞吐量：框架需支持每秒处理 1000 万条记录。
- 容错：任务失败率需 < 0.1%，支持自动重试与状态恢复。

**3) 核心架构/技术组件设计**：
- 调度层：使用分布式任务调度器（如 Spark Driver, Ray Head Node）分配任务给工作节点。
- 执行层：使用 Executor/Worker 节点进行数据处理，支持 CPU/GPU 混合调度。
- 存储层：与对象存储（S3）或分布式文件系统（HDFS）集成。
- 容错机制：基于 Checkpointing 和日志追加（Write-Ahead Log）实现状态恢复。

**4) 关键技术深入与可能解**：
- Spark vs Ray：
  - Apache Spark：基于 DAG 的批处理框架，擅长 SQL 和 ETL 任务，生态丰富，但 GPU 支持较弱。
  - Ray：原生支持 Python 和机器学习工作流，支持细粒度任务调度与 GPU 分配，适合 AI 训练与 RAG 管道。
  - Dask：类似 Spark 的 Python 分布式计算库，易于与 Pandas/NumPy 集成，但生态不如 Spark 成熟。

**5) Trade-off（权衡）分析**：
- Spark 的强一致性批处理 vs Ray 的灵活动态调度：Spark 适合固定 ETL 流程，Ray 适合动态 AI 工作流（如超参数搜索、分布式推理）。
- 内存管理：Spark 使用 Tungsten 内存管理，Ray 使用对象存储（Ray Object Store）共享大张量，减少序列化开销。

**6) 如何确定最优解**：
对于 AI 数据管道（尤其是涉及 Embedding 计算、向量检索、LLM 调用），Ray 是更优选择，因其原生支持 Python 和 GPU 调度；对于纯 ETL 和 SQL 转换，Spark 更合适。

**7) 名词和缩写全称及解释**：
- **DAG (Directed Acyclic Graph)**：有向无环图，表示任务依赖关系的图结构。
- **Checkpointing**：定期保存系统状态，以便在故障后恢复。
- **Ray Object Store**：Ray 框架中的分布式内存对象存储，用于高效共享大数据张量。
- **Tungsten**：Spark 的内存管理与代码生成引擎，优化 JVM 内存使用。

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
