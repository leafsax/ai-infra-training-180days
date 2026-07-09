## Day 7: ZeRO Optimization and Memory Management in Training

### 1) Topic and Core Examination Areas
- **Topic**: ZeRO (Zero Redundancy Optimizer) and Memory Management in Training.
- **Core Examination Areas**: ZeRO stages (1, 2, 3), optimizer state sharding, gradient sharding, and parameter sharding.

### 2) Requirement Clarification and Metric Definitions
- **Memory Breakdown in Training**: Parameters (~40%), Gradients (~40%), Optimizer States (~20% for Adam).
- **Target**: Fit a 70B model training within 80GB HBM per GPU using ZeRO-3.

### 3) Core Architecture/Technical Component Design
- **ZeRO-1**: Shards optimizer states across data parallel ranks.
- **ZeRO-2**: Adds gradient sharding.
- **ZeRO-3**: Adds parameter sharding; each GPU only stores the parameters it needs for the forward/backward pass.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **DeepSpeed ZeRO**: Microsoft's implementation of ZeRO optimizations.
- **Offloading**: Moving optimizer states or parameters to CPU memory or NVMe storage to further reduce HBM pressure.
- Comparative analysis: ZeRO-3 maximizes memory efficiency but increases communication overhead due to parameter gathering during forward/backward passes.

### 5) Trade-off analysis
- **Memory Efficiency vs. Communication Overhead**: ZeRO-3 significantly reduces per-GPU memory usage but requires more frequent cross-GPU communication to gather sharded parameters. Offloading to CPU/NVMe reduces HBM pressure but introduces CPU/memory bandwidth bottlenecks.

### 6) How to determine the optimal solution
- Start with ZeRO-2 for a good balance of memory savings and communication. Use ZeRO-3 when model parameters exceed GPU memory capacity. Use CPU/NVMe offloading only if HBM capacity is still insufficient and training speed can tolerate the overhead.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **ZeRO (Zero Redundancy Optimizer)**: A memory optimization technique that eliminates redundant copies of model states (parameters, gradients, optimizer states) across distributed training workers.
- **ZeRO-1/2/3**: Stages of ZeRO optimization, progressively sharding optimizer states, gradients, and parameters, respectively.
- **DeepSpeed**: A deep learning optimization library by Microsoft, famous for ZeRO and other memory/compute optimizations.
- **Optimizer States**: Variables maintained by the optimizer (e.g., momentum, variance in Adam) which can consume significant memory.

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
