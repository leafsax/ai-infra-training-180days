## Day 23: Embedding Serving and Similarity Search

### 1) Topic and Core Examination Areas
- **Topic**: Embedding Serving and Similarity Search Infrastructure.
- **Core Examination Areas**: High-throughput embedding model serving, batch embedding generation, and integration with vector search.

### 2) Requirement Clarification and Metric Definitions
- **Embedding Model Size**: e.g., 7B parameter embedding model or smaller (330M parameters).
- **Embedding Generation Throughput**: Target 10,000-50,000 documents/sec for ingestion pipelines.

### 3) Core Architecture/Technical Component Design
- **Embedding Serving Engine**: Similar to LLM serving but optimized for short text and high concurrency. Often uses the same engines (vLLM, TGI) but with different batching strategies.
- **Batch Embedding Generation**: Grouping documents into batches for efficient GPU utilization during embedding creation.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Embedding Model Optimization**: Using quantization (INT8/FP16) and optimized kernels for matrix multiplications in embedding layers.
- **Throughput vs. Latency in Embedding Serving**: Batching improves throughput for ingestion but may increase latency for real-time query embedding generation.

### 5) Trade-off analysis
- **Real-time vs. Batch Embedding**: Real-time embedding serving is needed for query embeddings in RAG but must be low-latency. Batch embedding is used for document ingestion and can be optimized for throughput rather than individual request latency.

### 6) How to determine the optimal solution
- Use high-throughput serving engines (vLLM, TensorRT-LLM) for embedding models. Separate infrastructure for batch document embedding (ingestion) and low-latency query embedding (serving) to optimize for their respective SLAs.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Embedding**: A dense vector representation of text, image, or other data, used to capture semantic meaning for similarity search.
- **Embedding Model**: A machine learning model specifically trained to generate embeddings for inputs like text or images.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Distributed Training)**:
  ```mermaid
  graph TD
      A[Data Loader] --> B[Training Nodes GPU+CPU]
      B --> C[NCCL All-Reduce Network]
      B --> D[Checkpoint Storage S3/NFS]
      B --> E[Model Registry MLflow]
      C --> B
  ```

- **Data Flow Diagram (Training)**:
  ```mermaid
  flowchart LR
      A[Training Data] --> B[Gradient Compute per GPU]
      B --> C[NCCL Gradient Sync]
      C --> D[Weight Update]
      D --> E[Checkpoint Save]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Inference Batching & Latency:** You mentioned Static vs. Continuous Batching. What is the exact kernel fusion overhead when dynamically splitting batches mid-generation? How do you guarantee TP99 latency when a long-context request (e.g., 128K tokens) arrives and forces KV cache eviction for shorter requests?
- **On Distributed Training Communication:** In DP/TP/PP, how do you handle straggler nodes in a 1024-GPU cluster? If one GPU is 5% slower, does the entire All-Reduce step block? What is the exact communication volume per step for NCCL All-Reduce with BF16 weights and gradients?
- **On Memory Optimization:** You cited ZeRO and PagedAttention. For ZeRO-3, how do you minimize the cross-node communication overhead during the optimizer step? For PagedAttention, what is the exact eviction policy (LRU vs. LFU vs. Attention-score-based) when HBM fragmentation occurs?
