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



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Data Pipeline)**:
  ```mermaid
  graph TD
      A[Data Sources] --> B[ETL/ELT Pipeline Spark/Flink]
      B --> C[Data Lake S3/Parquet]
      B --> D[Vector Database FAISS/Weaviate]
  ```

- **Data Flow Diagram (RAG System)**:
  ```mermaid
  flowchart LR
      A[User Query] --> B[RAG Orchestrator]
      B --> C[Vector DB Retrieval]
      C --> D[Context Construction]
      D --> E[LLM Inference]
      E --> F[Final Response]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On RAG Retrieval & Consistency:** In your RAG architecture, how do you handle embedding drift or vector database consistency during document updates? What is the exact latency budget allocated for the retrieval step vs. the LLM generation step to meet a TTFT < 200ms SLA?
- **On Data Pipeline Throughput:** You mentioned ETL/ELT pipelines with Spark/Flink. How do you prevent the data loader from becoming the bottleneck during training? What is the strategy for handling skewed data distributions that cause certain GPU workers to starve?
- **On Auto-Scaling Edge Cases:** For custom metrics-based scaling (e.g., KV cache utilization), what happens if the metrics server itself becomes unavailable? How do you design a fallback mechanism to prevent infinite scaling or sudden pod termination?
