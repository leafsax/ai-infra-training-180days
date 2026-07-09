## Day 56: Multi-Instance GPU (MIG) and GPU Partitioning for Multi-Tenancy

### 1) Topic and Core Examination Areas
**Topic**: GPU Virtualization and Multi-Tenancy.
**Core Examination Areas**: MIG (Multi-Instance GPU), GPU partitioning, and isolation for shared infrastructure.

### 2) Requirement Clarification and Metric Definitions
- **GPU Partitioning**: Splitting a single GPU into multiple isolated instances.
- **MIG Instances**: e.g., on an A100 80GB, MIG can create up to 7 instances of varying memory/compute configurations (e.g., 3g.20gb).
- **Tenancy Isolation**: Ensuring one tenant's workload does not impact another's performance or security.

### 3) Core Architecture/Technical Component Design
- **MIG Controller**: Manages the creation, configuration, and isolation of GPU instances.
- **Container Integration**: Kubernetes or Docker can schedule workloads on specific MIG instances.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **MIG (Multi-Instance GPU)**: Partitions a GPU into separate instances, each with its own memory, compute cores, and cache. Provides hardware-level isolation.
- **Time-Slicing**: Software-based GPU sharing where multiple containers share the same GPU by alternating execution time. Lower isolation, but more flexible.

### 5) Trade-off Analysis
- **MIG**: Strong hardware isolation, deterministic performance, but reduces flexibility (fixed partitions).
- **Time-Slicing**: Flexible resource sharing, but no hardware isolation and potential performance interference.

### 6) How to Determine the Optimal Solution
For multi-tenant cloud inference or fine-tuning services requiring strict SLAs and isolation, MIG is optimal. For internal shared development GPUs, time-slicing may suffice.

### 7) Glossary: Full Names and Explanations
- **MIG (Multi-Instance GPU)**: A technology (primarily on NVIDIA A100/H100 GPUs) that partitions a single GPU into multiple isolated instances, each with dedicated memory and compute resources.
- **Time-Slicing**: A GPU sharing technique where multiple workloads share a single GPU by taking turns using its compute resources in time intervals.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (GPU Cluster Management)**:
  ```mermaid
  graph TD
      A[Cluster Controller] --> B[GPU Nodes H100]
      B --> C[MIG / vGPU Partitioner]
      B --> D[RDMA Network Switch]
      A --> E[Auto-Scaler KEDA]
      E --> B
  ```

- **Data Flow Diagram (Scaling)**:
  ```mermaid
  flowchart LR
      A[Traffic Spike] --> B[Metrics Collector DCGM]
      B --> C[Scaler Controller]
      C --> D[Provision New GPU Pods]
      D --> E[Route Traffic to New Pods]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On KV Cache & PagedAttention:** You proposed PagedAttention for KV cache management. What happens when HBM is heavily fragmented under varying context lengths? How do you handle cache eviction policies under heavy load without degrading model accuracy or causing hallucinations?
- **On GPU Partitioning (MIG/vGPU):** For MIG or vGPU partitioning, what is the hard isolation guarantee? Can a noisy neighbor in one MIG partition still affect another via shared PCIe lanes or CPU memory bandwidth? How do you measure and enforce SLOs per partition in real-time?
- **On Auto-Scaling & Thrashing:** For KEDA-based auto-scaling, what is the scale-from-zero latency for bringing up a new vLLM pod? How do you prevent scaling thrashing when QPS fluctuates rapidly around the scaling threshold (e.g., +20% then -20% within 10 seconds)?
