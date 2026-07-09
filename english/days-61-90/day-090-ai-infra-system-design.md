### **Day 90: End-to-End AI Infra System Design: Integrating Data Pipelines, RAG, and Auto-Scaling**

**1) Topic and Core Examination Areas**
- Integrating data pipelines, RAG systems, and auto-scaling into a cohesive AI infrastructure
- End-to-end latency and cost optimization
- Production readiness and operational excellence

**2) Requirement Clarification and Metric Definitions**
- End-to-end RAG + Serving QPS: 10,000 QPS
- Data pipeline throughput: 10 TB/day ingestion
- RAG retrieval latency: P99 < 100 ms
- LLM serving TTFT: < 500 ms, P99 total latency < 3 seconds
- System availability: 99.99%

**3) Core Architecture/Technical Component Design**
- Data Pipeline: Kafka + Flink + Spark -> Feature Store / Vector DB ingestion
- RAG System: LlamaIndex/LangChain -> Vector DB (Milvus/Pinecone) -> Reranker -> LLM
- Serving & Auto-Scaling: vLLM/TensorRT-LLM on Kubernetes with KEDA auto-scaling based on TTFT and GPU utilization
- Observability: Prometheus + Grafana + LangSmith

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Integration Patterns**: Decouple ingestion (async) from serving (sync). Use event-driven architecture for data updates to vector DB. Use auto-scaling on serving layer to handle RAG query spikes.
- **End-to-End Optimization**: Cache embeddings and retrieval results; use continuous batching in vLLM; right-size GPU instances; implement rate limiting and backpressure.

**5) Trade-off analysis**
- Decoupled vs Monolithic: Decoupled pipelines and serving are more resilient and scalable but more complex to operate. Monolithic is simpler but harder to scale components independently.
- Caching vs Freshness: Caching improves latency and cost but may serve stale data; cache TTL must be tuned to data update frequency.

**6) How to determine the optimal solution**
- Design for decoupling: data ingestion (batch/stream) -> vector DB -> RAG orchestration -> LLM serving with auto-scaling.
- Implement caching at embedding and retrieval layers.
- Use KEDA + vLLM custom metrics (TTFT, GPU utilization) for auto-scaling.
- Ensure observability and rate limiting for production readiness.

**7) Full names and explanations of nouns and abbreviations**
- **Review of key terms from Days 61-90**: RAG, TTFT, TPOT, QPS, TPS, P99, HPA, VPA, KEDA, HNSW, FAISS, BM25, RRF, HyDE, PII, GDPR, HIPAA, vLLM, TensorRT-LLM, KServe, PagedAttention, KV Cache, HBM, DCGM, nvml, Karpenter, Spot Instances, L4/L7 Load Balancing, APM, OpenTelemetry, LangSmith, etc.

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
