### Day 65: 特征存储与向量数据管道 (Feature Store and Vector Data Pipeline)

**1) 题目与考察核心**：
设计一个支持实时特征提取与向量存储的管道，用于 RAG 系统和推荐模型。考察向量嵌入（Embedding）生成、特征存储架构与实时/离线一致性。

**2) 需求澄清与指标定义**：
- 向量维度：768 维（如 using text-embedding-3-small）。
- 更新频率：实时数据流入，向量更新延迟 < 5 秒。
- 查询延迟：向量相似度搜索（Similarity Search）P99 延迟 < 50 ms。
- 吞吐量：每秒处理 10,000 次向量插入/更新，100,000 次相似度查询。

**3) 核心架构/技术组件设计**：
- 特征提取层：使用 Embedding 模型（如 OpenAI API 或本地部署的 Sentence-Transformers）将文本转换为向量。
- 特征存储层：使用 Feature Store（如 Feast）管理离线/在线特征。
- 向量数据库层：使用 Milvus, Pinecone 或 FAISS 存储向量索引，支持 HNSW 算法。

**4) 关键技术深入与可能解**：
- 向量索引算法对比：
  - HNSW (Hierarchical Navigable Small World)：图-based 索引，查询速度快，精度高，但内存占用大。
  - IVF (Inverted File Index)：基于聚类的索引，存储效率高，但查询速度略慢。
  - FAISS vs 云原生向量库：FAISS 适合本地部署与自定义索引，Pinecone/Milvus 提供托管服务与自动扩缩容。

**5) Trade-off（权衡）分析**：
- HNSW vs IVF：HNSW 延迟低（<10ms），但 HBM/内存消耗大；IVF 内存友好，但 P99 延迟可能达到 50-100ms。
- 实时 Embedding 生成 vs 预计算：实时生成灵活但增加推理延迟；预计算存储向量但更新滞后。

**6) 如何确定最优解**：
对于 RAG 系统，采用 HNSW 索引的 Milvus 或 Pinecone，结合 Feast Feature Store 管理元数据，Embedding 生成采用异步队列（Kafka）+ 批量 Embedding 模型推理。

**7) 名词和缩写全称及解释**：
- **Feature Store**：特征存储，用于管理和分发机器学习特征的系统。
- **HNSW (Hierarchical Navigable Small World)**：层次可导航小世界图算法，用于高效近似最近邻搜索。
- **IVF (Inverted File Index)**：倒排文件索引，一种基于聚类的前置过滤向量检索方法。
- **FAISS (Facebook AI Similarity Search)**：Facebook 开源的向量相似度搜索库。

---

