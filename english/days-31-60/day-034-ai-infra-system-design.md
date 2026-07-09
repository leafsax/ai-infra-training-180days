## Day 34: LLM Serving - Data Parallelism (DP) and Distributed Inference

### 1) Topic and Core Examination Areas
**Topic**: Data Parallelism for Inference and Distributed Serving Architectures.
**Core Examination Areas**: How Data Parallelism (DP) applies to inference (Model Parallelism + Data Parallelism), and distributed routing strategies.

### 2) Requirement Clarification and Metric Definitions
- **DP Degree**: Number of replicas of the model across different GPU groups (e.g., DP=2 means 2 full copies of the model on 2x TP=4 groups).
- **Load Balancer**: Component that routes incoming requests to different model replicas.
- **QPS per Replica**: Expected query load handled by each model instance.

### 3) Core Architecture/Technical Component Design
- **Model Replicas**: Multiple identical copies of the model, each capable of serving requests independently.
- **Request Router/Distributor**: Uses strategies like Round-Robin, Least-Connections, or KV-cache-aware routing to distribute requests to replicas.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Naive DP (Model Replication)**: Each replica has a full copy of the model. Simple load balancing (Round-Robin) works well if requests are similar in length.
- **KV-Aware DP Routing**: Routes requests to the replica with the most available KV cache capacity or similar sequence lengths to optimize batching efficiency.

### 5) Trade-off Analysis
- **Naive DP**: Simple to implement, but may lead to unbalanced KV cache usage and suboptimal batching across replicas.
- **KV-Aware Routing**: Complex to implement, requires state sharing among replicas, but maximizes overall system throughput.

### 6) How to Determine the Optimal Solution
For high-QPS production systems with variable request lengths, KV-aware routing or sequence-length-aware load balancing is optimal to ensure each DP replica achieves high continuous batching efficiency.

### 7) Glossary: Full Names and Explanations
- **Data Parallelism (DP)**: A distributed training/serve technique where full copies of the model are placed on different devices, and each device processes a different subset of the data (or in inference, handles a shard of the request traffic).
- **Load Balancer**: A system or component that distributes incoming network traffic across multiple servers or model replicas to ensure no single resource is overwhelmed.
- **KV-Aware Routing**: A load balancing strategy that considers the state of the KV cache across model replicas to make routing decisions that optimize batching and memory usage.

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
