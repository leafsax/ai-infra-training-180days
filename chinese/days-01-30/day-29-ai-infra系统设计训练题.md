## 第29天：RAG Infra系统架构（Embedding服务与向量数据库）

### 1) 题目与考察核心
**题目**：设计支持RAG（检索增强生成）的完整Infra系统，包括Embedding服务和Vector DB。
**考察核心**：Embedding模型Serving，向量数据库（Vector DB）架构，Chunking策略。

### 2) 需求澄清与指标定义
- **向量库规模**：10亿向量，维度1536。
- **指标**：Embedding生成延迟 < 100ms，Vector Search（Top-K）延迟 < 50ms，吞吐 > 1000 QPS。

### 3) 核心架构/技术组件设计
- **Embedding Serving Engine**：使用专门模型（如text-embedding-3-small）的批量推理服务。
- **Vector Database**：如Milvus, Qdrant, Pinecone，支持HNSW（Hierarchical Navigable Small World）算法。

### 4) 关键技术深入与可能解
- *HNSW Index*：向量搜索的图索引算法，平衡搜索精度与速度。
- *Chunking & Metadata*：文档切块策略及元数据过滤，提升检索相关性。

### 5) Trade-off（权衡）分析
- *精确搜索 vs 近似搜索*：精确搜索（Brute Force）精度高但慢；HNSW近似搜索速度快但需调优epsilon/efSearch参数以平衡精度。

### 6) 如何确定最优解
- 采用Milvus或Qdrant部署HNSW索引，Embedding服务采用vLLM或TextGen Inference进行批量推理，结合元数据过滤提升RAG准确率。

### 7) 名词和缩写全称及解释
- **RAG (Retrieval-Augmented Generation)**：检索增强生成，结合外部知识库和LLM生成回答。
- **Embedding**：将文本转换为高维向量的技术。
- **Vector DB (Vector Database)**：向量数据库，专门存储和检索高维向量的数据库。
- **HNSW (Hierarchical Navigable Small World)**：用于近似最近邻搜索的图算法。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 分布式训练）**：
  ```mermaid
  graph TD
      A[数据加载器] --> B[训练节点 GPU+CPU]
      B --> C[NCCL All-Reduce 网络]
      B --> D[检查点存储 S3/NFS]
      B --> E[模型注册表 MLflow]
      C --> B
  ```

- **数据流图（Data Flow Diagram - 训练流程）**：
  ```mermaid
  flowchart LR
      A[训练数据] --> B[各GPU梯度计算]
      B --> C[NCCL梯度同步]
      C --> D[权重更新]
      D --> E[检查点保存]
  ```
