## Day 16: GPU Virtualization and Multi-tenant Serving

### 1) Topic and Core Examination Areas
- **Topic**: GPU Virtualization and Multi-tenant Inference.
- **Core Examination Areas**: Time-slicing, MIG (Multi-Instance GPU), and resource isolation for serving multiple models or users on the same GPU.

### 2) Requirement Clarification and Metric Definitions
- **Tenant Isolation**: Ensure one tenant's workload does not degrade another's TTFT or throughput.
- **GPU Partitioning**: e.g., Split an 80GB H100 into 7 instances of ~11GB each (MIG) or use time-slicing with 10ms timequanta.

### 3) Core Architecture/Technical Component Design
- **MIG (Multi-Instance GPU)**: NVIDIA's feature to partition a GPU into separate instances, each with its own compute, memory, and cache resources.
- **Time-Slicing**: A software-based approach where the GPU scheduler rapidly switches between different workloads (tenants) using context switching.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **MIG vs. Time-Slicing**: MIG provides hard isolation and is suitable for different models with static resource needs. Time-slicing is more flexible but can suffer from context-switching overhead and lack of strict memory isolation.
- **vGPU (Virtual GPU)**: VMware/NVIDIA technology for virtualizing GPUs in virtualized environments.

### 5) Trade-off analysis
- **Isolation vs. Utilization**: MIG offers strong isolation but can lead to fragmented, underutilized GPU resources if partitions are not fully used. Time-slicing maximizes utilization but offers weaker isolation and potential latency spikes.

### 6) How to determine the optimal solution
- For multi-tenant serving with strict SLA requirements, use MIG or dedicated GPU instances per tenant. For internal workloads with flexible SLAs, use time-slicing within a serving engine that supports multi-model batching or request prioritization.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **GPU Virtualization**: Techniques to share a single physical GPU among multiple users or workloads.
- **MIG (Multi-Instance GPU)**: An NVIDIA GPU feature that partitions a GPU into multiple independent instances, each with dedicated compute, memory, and cache.
- **Time-Slicing**: A scheduling technique where GPU time is divided into small quanta and shared among multiple workloads via rapid context switching.
- **SLA (Service Level Agreement)**: A commitment between a service provider and a client regarding service quality, such as latency or availability.

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
