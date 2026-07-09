## Day 32: LLM Serving - KV Cache Management and PagedAttention

### 1) Topic and Core Examination Areas
**Topic**: KV Cache Management and PagedAttention Mechanism.
**Core Examination Areas**: Understanding what KV Cache is, why it is a memory bottleneck, and how PagedAttention solves memory fragmentation and overhead issues.

### 2) Requirement Clarification and Metric Definitions
- **KV Cache Size**: For a 70B model with 8x H100 (80GB HBM), the KV cache can consume 30-40GB of memory for moderate batch sizes.
- **Memory Fragmentation**: Wasted memory due to non-contiguous KV cache allocations.
- **Context Window**: The maximum number of tokens a model can process at once (e.g., 128K tokens).

### 3) Core Architecture/Technical Component Design
- **KV Cache Buffer**: A dedicated memory region on the GPU to store Key and Value tensors for each token in the sequence.
- **Memory Manager**: Allocates and deallocates KV cache blocks dynamically.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Naive KV Cache Allocation**: Allocates contiguous memory for each sequence. Leads to severe fragmentation and limits batch size.
- **PagedAttention**: Inspired by virtual memory paging in operating systems. Divides the KV cache into fixed-size blocks. Blocks can be non-contiguous in memory but are mapped logically via a page table. This eliminates fragmentation and allows dynamic sharing of KV cache blocks (e.g., in forked sequences like in batched sampling).

### 5) Trade-off Analysis
- **Naive Allocation**: Simple but wastes memory, limiting throughput.
- **PagedAttention**: High memory efficiency, supports larger batch sizes and longer context windows, but adds slight overhead for page table lookups.

### 6) How to Determine the Optimal Solution
For any production LLM serving system supporting long contexts and high concurrency, PagedAttention (or an equivalent block-wise KV cache management system) is mandatory.

### 7) Glossary: Full Names and Explanations
- **KV Cache (Key-Value Cache)**: In transformer models, the cached Key and Value tensors from previous decoding steps to avoid recomputing them, significantly speeding up inference.
- **Context Window**: The maximum number of tokens a model can take as input at once.
- **PagedAttention**: An attention algorithm that uses concepts from virtual memory and paging systems to efficiently manage KV cache memory, eliminating fragmentation and enabling dynamic allocation.
- **Memory Fragmentation**: A condition where memory is allocated in non-contiguous blocks, leading to wasted space and inability to allocate large contiguous segments.

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
