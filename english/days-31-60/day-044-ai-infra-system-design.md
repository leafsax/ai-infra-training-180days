## Day 44: Retrieval-Augmented Generation (RAG) System Architecture and Infra

### 1) Topic and Core Examination Areas
**Topic**: RAG System Infrastructure and Design.
**Core Examination Areas**: Document ingestion, embedding generation, vector storage, retrieval, and LLM generation integration.

### 2) Requirement Clarification and Metric Definitions
- **Document Volume**: e.g., 10 million documents or PDFs.
- **Embedding Dimension**: e.g., 1536 dimensions for OpenAI text-embedding-3-small.
- **Retrieval Latency**: Target < 100ms for vector search.
- **QPS for Embedding Service**: e.g., 50 QPS for document ingestion.

### 3) Core Architecture/Technical Component Design
- **Ingestion Pipeline**: Text extraction (PDFs, HTML), chunking, embedding generation, and vector storage.
- **Retrieval Service**: Vector database query engine.
- **Generation Service**: LLM serving engine that incorporates retrieved context into the prompt.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Chunking Strategies**: Fixed-size, semantic, or recursive chunking to preserve context.
- **Embedding Models**: Dedicated small models (e.g., BGE, text-embedding-ada) for generating vector representations.
- **Hybrid Search**: Combining vector similarity search with keyword search (BM25) for better retrieval accuracy.

### 5) Trade-off Analysis
- **Vector-Only Search**: Good for semantic similarity, but may miss exact keyword matches.
- **Hybrid Search (Vector + BM25)**: Higher accuracy, but more complex infrastructure and query routing.

### 6) How to Determine the Optimal Solution
For enterprise RAG systems where accuracy is critical, hybrid search (vector + keyword) with semantic chunking and a dedicated embedding serving infrastructure is optimal.

### 7) Glossary: Full Names and Explanations
- **RAG (Retrieval-Augmented Generation)**: An architecture where an LLM's generation is augmented by retrieving relevant information from an external knowledge base before generating a response.
- **Embedding**: A dense vector representation of text (or other data) that captures semantic meaning, used for similarity search.
- **Vector Database**: A specialized database optimized for storing and querying high-dimensional vectors, used for similarity search in RAG systems.
- **BM25**: A ranking function used by search engines to estimate the relevance of documents to a given search query.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram**:
  ```mermaid
  graph TD
      A[Load Balancer] --> B[Inference Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU Resource Manager]
      D --> E[Kubernetes / Karpenter]
      B --> F[Telemetry DCGM/OpenTelemetry]
  ```

- **Data Flow Diagram**:
  ```mermaid
  flowchart LR
      A[Incoming Requests] --> B[Continuous Batching Scheduler]
      B --> C[KV Cache Lookup & Update]
      C --> D[GPU Tensor Core Compute]
      D --> E[Output Tokens]
      E --> F[Streaming Response]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On KV Cache & PagedAttention:** You proposed PagedAttention for KV cache management. What happens when HBM is heavily fragmented under varying context lengths? How do you handle cache eviction policies under heavy load without degrading model accuracy or causing hallucinations?
- **On GPU Partitioning (MIG/vGPU):** For MIG or vGPU partitioning, what is the hard isolation guarantee? Can a noisy neighbor in one MIG partition still affect another via shared PCIe lanes or CPU memory bandwidth? How do you measure and enforce SLOs per partition in real-time?
- **On Auto-Scaling & Thrashing:** For KEDA-based auto-scaling, what is the scale-from-zero latency for bringing up a new vLLM pod? How do you prevent scaling thrashing when QPS fluctuates rapidly around the scaling threshold (e.g., +20% then -20% within 10 seconds)?
