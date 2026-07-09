## Day 45: Vector Databases and Embedding Serving Infrastructure

### 1) Topic and Core Examination Areas
**Topic**: Vector Database Infrastructure and Embedding Service Scaling.
**Core Examination Areas**: Vector DB architectures (HNSW, IVF), embedding model serving, and throughput/latency trade-offs.

### 2) Requirement Clarification and Metric Definitions
- **Vector Dimension**: e.g., 1536 or 3072.
- **HNSW (Hierarchical Navigable Small World)**: A graph-based indexing algorithm for approximate nearest neighbor (ANN) search.
- **Latency Target**: p99 vector search latency < 50ms for 10M vectors.

### 3) Core Architecture/Technical Component Design
- **Embedding Serving Engine**: Serves the embedding model (e.g., using vLLM or TorchServe) to convert text chunks to vectors.
- **Vector DB Cluster**: Distributes vector indices and data across multiple nodes for horizontal scaling.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **HNSW Index**: Provides high recall and low latency for ANN search but requires significant memory (10-20x the size of raw vectors).
- **IVF (Inverted File Index)**: Clusters vectors into centroids; search is performed within relevant clusters. More memory-efficient than HNSW but higher latency.

### 5) Trade-off Analysis
- **HNSW**: Best latency and recall, but high memory cost.
- **IVF**: Lower memory usage, but higher latency and potentially lower recall.

### 6) How to Determine the Optimal Solution
For RAG systems requiring sub-100ms latency and high recall with available memory, HNSW is optimal. For very large vector sets with strict memory constraints, IVF or product quantization is preferred.

### 7) Glossary: Full Names and Explanations
- **HNSW (Hierarchical Navigable Small World)**: A graph-based algorithm for approximate nearest neighbor search that provides fast and accurate vector similarity search.
- **IVF (Inverted File Index)**: A vector indexing method that partitions vectors into clusters (centroids) and searches only within the nearest clusters to reduce computation.
- **ANN (Approximate Nearest Neighbor)**: A search method that finds vectors similar to a query vector with high probability, but not guaranteed to be the absolute closest.

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
