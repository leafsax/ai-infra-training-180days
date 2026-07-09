### **Day 69: Advanced Retrieval: Hybrid Search, Reranking, and Query Expansion**

**1) Topic and Core Examination Areas**
- Hybrid search (vector + keyword/lexical search)
- Reranking models and two-stage retrieval
- Query expansion and transformation

**2) Requirement Clarification and Metric Definitions**
- Retrieval recall target: Top-5 recall > 85%
- Reranking latency: P99 < 200 ms for reranking top-50 candidates
- Hybrid search QPS: 5,000 QPS
- Precision@K: Precision at K=5 should be > 70%

**3) Core Architecture/Technical Component Design**
- First-stage retrieval: Vector search + Keyword search (BM25) combined via weighted scoring or Reciprocal Rank Fusion (RRF)
- Reranking stage: Cross-encoder models (e.g., Cross-Encoder NLLB, RankLLM, Cohere Rerank) to score top-K candidates
- Query Expansion: Use LLM to expand query with synonyms, or use HyDE (Hypothetical Document Embeddings)

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Vector vs Keyword Search**: Vector search captures semantic similarity but may miss exact keyword matches. Keyword search (BM25) excels at exact term matching but fails on synonyms or semantic variation. Hybrid search combines both.
- **Cross-encoder vs Bi-encoder Reranking**: Bi-encoders encode query and document separately (fast, used in first-stage). Cross-encoders encode query+document together (accurate but slower, used for reranking top-K).

**5) Trade-off analysis**
- Hybrid Search: Improves recall and precision but adds complexity and compute (maintaining keyword index like Elasticsearch).
- Reranking: Significantly improves final answer quality but adds latency (200-500 ms per query).
- Query Expansion: Can improve retrieval for vague queries but may introduce noise or hallucinated terms.

**6) How to determine the optimal solution**
- For general RAG systems, implement hybrid search (vector + BM25) with RRF.
- If answer quality is critical and latency budget allows (>500 ms total retrieval), add a cross-encoder reranking stage for top-50 candidates.
- For user queries that are short or ambiguous, use query expansion or HyDE.

**7) Full names and explanations of nouns and abbreviations**
- **BM25**: Best Matching 25. A ranking function used by search engines to estimate the relevance of documents to a given search query.
- **RRF**: Reciprocal Rank Fusion. A method to combine results from multiple search strategies by fusing their ranked lists.
- **HyDE**: Hypothetical Document Embeddings. A query expansion technique that generates a hypothetical document for a query and uses its embedding for search.
- **Precision@K**: The fraction of relevant items among the top K retrieved items.
- **Recall**: The fraction of relevant items that are retrieved among all relevant items in the corpus.

---

