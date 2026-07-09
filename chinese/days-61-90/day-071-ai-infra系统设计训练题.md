### Day 71: RAG 系统架构概述 (RAG System Architecture Overview)

**1) 题目与考察核心**：
设计一个完整的 RAG（Retrieval-Augmented Generation）系统架构，涵盖索引、检索与生成阶段。

**2) 需求澄清与指标定义**：
- QPS：1000 QPS 检索请求，200 QPS 生成请求。
- 延迟指标：检索 P99 延迟 < 100 ms，生成 TTFT < 500 ms，总响应时间 TP99 < 3 秒。
- 准确率：检索命中率（Hit Rate@5）> 85%。

**3) 核心架构/技术组件设计**：
- 索引管道：文档解析、分块、Embedding、向量存储。
- 检索管道：查询编码、向量相似度搜索、重排序（Re-ranking）。
- 生成管道：LLM 推理服务（如 vLLM/TGI），上下文拼接与生成。

**4) 关键技术深入与可能解**：
- 检索策略：稀疏检索（BM25）vs 密集检索（Dense Retrieval）。BM25 对关键词匹配好，Dense 对语义匹配好。

**5) Trade-off（权衡）分析**：
- 单一检索 vs 混合检索：混合检索（Hybrid Search）精度高但延迟与成本高；单一检索简单但可能遗漏相关信息。

**6) 如何确定最优解**：
采用 Hybrid Search（BM25 + Dense）+ RRF（Reciprocal Rank Fusion）+ Re-ranker 的三级检索架构。

**7) 名词和缩写全称及解释**：
- **RAG (Retrieval-Augmented Generation)**：检索增强生成，结合外部知识库与 LLM 生成回答。
- **TTFT (Time to First Token)**：首个 token 生成时间，衡量 LLM 响应延迟的关键指标。
- **TP99**：99% 的请求延迟低于该值。
- **Hit Rate@K**：前 K 个检索结果中包含相关文档的比例。
- **BM25**：一种基于词频的文本检索算法。
- **RRF (Reciprocal Rank Fusion)**：一种融合多个检索结果排名的算法。

---

