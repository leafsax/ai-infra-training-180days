## Day 5: Distributed Training Fundamentals - Data Parallelism

### 1) Topic and Core Examination Areas
- **Topic**: Distributed Training Fundamentals - Data Parallelism (DP).
- **Core Examination Areas**: How data parallelism works, gradient synchronization, and performance bottlenecks in DP training.

### 2) Requirement Clarification and Metric Definitions
- **Batch Size per GPU (Microbatch)**: e.g., 32 sequences per GPU.
- **Global Batch Size**: Microbatch size × Number of GPUs.
- **Synchronization Latency**: Time taken for All-Reduce operations to synchronize gradients across GPUs.

### 3) Core Architecture/Technical Component Design
- **Data Parallelism Architecture**: Each GPU holds a full copy of the model. Data is sharded across GPUs. Each GPU computes gradients on its shard, and gradients are synchronized via All-Reduce.
- **Distributed Data Parallel (DDP)**: A PyTorch mechanism for implementing data parallelism.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Gradient Synchronization**: Uses All-Reduce operations (implemented via NCCL - NVIDIA Collective Communications Library).
- **Gradient Accumulation**: Used when the global batch size is too large to fit in memory; gradients are accumulated over several micro-batches before synchronization.

### 5) Trade-off analysis
- **Communication vs. Computation Overlap**: In DP, communication (gradient sync) can become the bottleneck if the model is small or the network is slow. Overlapping communication with computation (via pipelining or asynchronous updates) can mitigate this.

### 6) How to determine the optimal solution
- For models that fit on a single GPU, start with Data Parallelism (DDP). Ensure the network interconnect (InfiniBand/RoCE) has sufficient bandwidth to handle the All-Reduce traffic for the chosen global batch size and model gradient size.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Data Parallelism (DP)**: A distributed training strategy where each worker holds a copy of the model and processes a different shard of the data.
- **Gradient Synchronization**: The process of aggregating gradients computed by different GPUs to ensure model consistency.
- **All-Reduce**: A collective communication operation that computes a reduction (e.g., sum) of data across all processes and distributes the result to all.
- **DDP (Distributed Data Parallel)**: A PyTorch module for performing data-parallel training.
- **NCCL (NVIDIA Collective Communications Library)**: A library providing standard collective communication operations (All-Reduce, Broadcast, etc.) optimized for NVIDIA GPUs.
- **Microbatch**: A smaller batch of data processed by a single GPU before gradient accumulation or synchronization.

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


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Inference Batching & Latency:** You mentioned Static vs. Continuous Batching. What is the exact kernel fusion overhead when dynamically splitting batches mid-generation? How do you guarantee TP99 latency when a long-context request (e.g., 128K tokens) arrives and forces KV cache eviction for shorter requests?
- **On Distributed Training Communication:** In DP/TP/PP, how do you handle straggler nodes in a 1024-GPU cluster? If one GPU is 5% slower, does the entire All-Reduce step block? What is the exact communication volume per step for NCCL All-Reduce with BF16 weights and gradients?
- **On Memory Optimization:** You cited ZeRO and PagedAttention. For ZeRO-3, how do you minimize the cross-node communication overhead during the optimizer step? For PagedAttention, what is the exact eviction policy (LRU vs. LFU vs. Attention-score-based) when HBM fragmentation occurs?
