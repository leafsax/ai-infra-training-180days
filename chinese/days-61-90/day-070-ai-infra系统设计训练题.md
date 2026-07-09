### Day 70: 数据管道监控与质量保证 (Data Pipeline Monitoring and Quality Assurance)

**1) 题目与考察核心**：
设计数据管道监控与质量评估系统，检测数据漂移（Data Drift）、质量下降与管道故障。

**2) 需求澄清与指标定义**：
- 监控延迟：指标计算与告警延迟 < 1 分钟。
- 数据漂移检测：检测分布变化（KL 散度 > 阈值）并触发告警。

**3) 核心架构/技术组件设计**：
- 指标收集：使用 Prometheus + Grafana 监控管道吞吐量、延迟、错误率。
- 数据质量检查：使用 Great Expectations 或 Deequ 定义数据质量规则。
- 漂移检测：使用 Evidently AI 或自定义统计测试检测特征分布变化。

**4) 关键技术深入与可能解**：
- 统计漂移检测 vs 模型驱动检测：统计方法（如 KS 测试）计算快但仅限数值特征；模型驱动（如使用分类器区分训练/生产数据）更通用但成本高。

**5) Trade-off（权衡）分析**：
- 实时监控 vs 离线分析：实时监控响应快但资源消耗大；离线分析成本低但延迟高。

**6) 如何确定最优解**：
采用分层监控：基础指标（吞吐量、错误率）实时 Prometheus 监控；数据质量与漂移使用每日离线 Great Expectations 检查。

**7) 名词和缩写全称及解释**：
- **Data Drift**：数据漂移，指生产数据分布与训练数据分布发生变化。
- **KL Divergence (Kullback-Leibler Divergence)**：KL 散度，衡量两个概率分布差异的统计量。
- **Great Expectations**：开源数据质量验证工具。

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
