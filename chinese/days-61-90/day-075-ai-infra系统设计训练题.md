### Day 75: 检索策略：稀疏 vs 密集 vs 混合 (Retrieval Strategies: Sparse vs Dense vs Hybrid)

**1) 题目与考察核心**：
比较并设计稀疏检索（BM25）、密集检索（Dense Retrieval）与混合检索（Hybrid Search）架构。

**2) 需求澄清与指标定义**：
- 查询类型：关键词查询 vs 语义查询。
- 混合检索融合：RRF 或 Learn-to-Rank。

**3) 核心架构/技术组件设计**：
- 稀疏索引：Inverted Index（倒排索引）。
- 密集索引：HNSW 向量索引。
- 融合层：RRF 算法合并排名。

**4) 关键技术深入与可能解**：
- BM25 vs Dense：BM25 对专有名词匹配好，Dense 对语义泛化好。

**5) Trade-off（权衡）分析**：
- 混合检索增加架构复杂度但显著提升 Hit Rate。

**6) 如何确定最优解**：
采用 Hybrid Search + RRF + Cross-Encoder Re-ranker 的三级架构。

**7) 名词和缩写全称及解释**：
- **Cross-Encoder Re-ranker**：将查询与文档共同输入模型进行精细评分的重排序模型。

---

