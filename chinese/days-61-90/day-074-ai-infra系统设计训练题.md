### Day 74: Embedding 模型与表示学习 (Embedding Models and Representation Learning)

**1) 题目与考察核心**：
选型与微调 Embedding 模型，考察模型能力、维度选择与部署优化。

**2) 需求澄清与指标定义**：
- 模型维度：384, 768, 1536。
- 推理延迟：单请求 < 10 ms（CPU）或 < 2 ms（GPU）。
- 准确率：MTEB 基准测试 Rank > Top 10。

**3) 核心架构/技术组件设计**：
- 模型选型：OpenAI text-embedding-3-small, BGE-m3, 或自训练模型。
- 部署：使用 TensorRT-LLM 或 ONNX Runtime 加速。

**4) 关键技术深入与可能解**：
- 通用 Embedding vs 领域微调：通用模型覆盖广，领域微调提升特定场景准确率但需标注数据。

**5) Trade-off（权衡）分析**：
- 维度大小：高维度精度高但存储与计算成本高；低维度高效但精度略降。

**6) 如何确定最优解**：
采用 BGE-m3 或 text-embedding-3-small，使用 ONNX 部署于 CPU/GPU 混合集群，平衡延迟与成本。

**7) 名词和缩写全称及解释**：
- **Embedding**：将离散对象（如文本）映射为连续向量的技术。
- **MTEB (Massive Text Embedding Benchmark)**：文本嵌入模型基准测试。
- **TensorRT-LLM**：NVIDIA 的 LLM 推理优化库。

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
