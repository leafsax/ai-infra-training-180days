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

