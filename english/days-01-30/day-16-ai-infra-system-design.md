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

