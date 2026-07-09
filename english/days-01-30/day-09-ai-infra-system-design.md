## Day 9: Batching Strategies - Static vs Dynamic vs Continuous Batching

### 1) Topic and Core Examination Areas
- **Topic**: Batching Strategies for LLM Inference.
- **Core Examination Areas**: Static Batching, Dynamic Batching, Continuous Batching (In-flight Batching), and their impact on throughput and latency.

### 2) Requirement Clarification and Metric Definitions
- **Batch Size**: Number of requests processed simultaneously. Target dynamic batch size 16-64 depending on context length and HBM.
- **Latency vs. Throughput Trade-off**: Larger batches increase throughput but may increase TTFT and ITL for individual requests.

### 3) Core Architecture/Technical Component Design
- **Batching Manager**: Collects incoming requests and forms batches for the model executor.
- **Request Queue**: Buffers incoming requests waiting to be batched.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Static Batching**: Fixed batch size; all requests in the batch must have similar completion times. Inefficient for varying context lengths.
- **Dynamic Batching**: Requests are batched together as they arrive, up to a maximum batch size. Improves utilization but can increase latency for late-arriving requests.
- **Continuous Batching (In-flight Batching)**: Requests are removed from the batch as soon as they complete (reach EOS), and new requests are injected immediately. Maximizes GPU utilization.

### 5) Trade-off analysis
- **Latency vs. Throughput**: Static batching offers predictable latency but low throughput. Dynamic batching improves throughput at the cost of variable latency. Continuous batching maximizes throughput while minimizing idle GPU time, but requires complex scheduler logic.

### 6) How to determine the optimal solution
- For production LLM serving with varying request lengths and QPS, Continuous Batching (as implemented in vLLM) is the optimal choice to maximize throughput and GPU utilization while keeping TTFT reasonable.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Batching**: The process of combining multiple requests into a single group to be processed together by the model, improving hardware utilization.
- **Static Batching**: A batching strategy where the batch size is fixed and all requests are processed for the same number of steps.
- **Dynamic Batching**: A strategy where requests are grouped into batches dynamically as they arrive, up to a maximum size or latency timeout.
- **Continuous Batching (In-flight Batching)**: A batching strategy where completed requests are immediately removed from the batch and new requests are added, allowing for continuous GPU utilization.
- **EOS (End of Sequence)**: A special token indicating the end of a generated sequence.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram**:
  ```mermaid
  graph TD
      A[Client/User] --> B[API Gateway / Load Balancer]
      B --> C[vLLM/TGI Inference Engine]
      C --> D[GPU Cluster H100/A100]
      C --> E[Model Weights Storage NVMe/S3]
      C --> F[KV Cache Manager]
      D --> G[GPU Monitoring DCGM]
  ```

- **Data Flow Diagram**:
  ```mermaid
  flowchart LR
      A[User Prompt] --> B[API Gateway]
      B --> C[Request Queue & Batching]
      C --> D[GPU Inference Compute]
      D --> E[Token Generation]
      E --> F[Response Stream]
  ```
