## Day 4: Network Interconnects for AI Clusters

### 1) Topic and Core Examination Areas
- **Topic**: Network Interconnects for AI Clusters.
- **Core Examination Areas**: InfiniBand, Ethernet (RoCE v2), network topology (Fat-Tree, Dragonfly), and network bandwidth/latency requirements for distributed training and serving.

### 2) Requirement Clarification and Metric Definitions
- **Network Bandwidth**: Target 400 Gbps or 800 Gbps per node for training clusters.
- **Network Latency**: Target < 1 microsecond for InfiniBand RDMA operations.
- **All-Reduce Performance**: Critical metric for distributed training synchronization.

### 3) Core Architecture/Technical Component Design
- **Network Topology**: Fat-Tree topology for uniform bandwidth between any two nodes; Dragonfly for scaling to thousands of nodes.
- **Interconnect Technologies**: InfiniBand (proprietary, low latency, high bandwidth) vs. Ethernet with RoCE v2 (RDMA over Converged Ethernet).

### 4) Deep Dive into Key Technologies and Possible Solutions
- **InfiniBand vs. RoCE v2**: InfiniBand offers native RDMA, low latency, and lossless networking (via FCV). RoCE v2 runs RDMA over standard Ethernet, requiring DCQCN or PFC for lossless networking.
- **RDMA (Remote Direct Memory Access)**: Allows data to be sent directly from the memory of one computer to another without involving the CPU of either.

### 5) Trade-off analysis
- **InfiniBand vs. Ethernet/RoCE**: InfiniBand offers superior out-of-the-box performance and lossless networking but is more expensive and vendor-locked (Cisco/Mellanox). RoCE v2 uses commodity Ethernet hardware, reducing cost but requiring careful network tuning (PFC, ECN) to avoid congestion and packet loss.

### 6) How to determine the optimal solution
- For large-scale distributed training (1000+ GPUs), InfiniBand is often preferred for its proven low-latency, lossless properties. For serving clusters or smaller training clusters, high-speed Ethernet (200G/400G) with RoCE v2 or TCP/IP is often sufficient and more cost-effective.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **InfiniBand**: A high-performance networking technology used in high-performance computing (HPC) and AI clusters, supporting RDMA.
- **RoCE v2 (RDMA over Converged Ethernet version 2)**: A protocol that allows RDMA operations to be carried over IP networks (Ethernet).
- **RDMA (Remote Direct Memory Access)**: A network technology that enables data exchange between computer memories without CPU involvement.
- **Fat-Tree Topology**: A network topology designed to provide high bandwidth and multiple paths between any two nodes, minimizing congestion.
- **All-Reduce**: A distributed computing operation where all nodes contribute data, the data is aggregated (e.g., summed), and the result is distributed to all nodes.
- **PFC (Priority-based Flow Control)**: A link-level flow control mechanism in Ethernet to prevent packet loss due to congestion.
- **ECN (Explicit Congestion Notification)**: A network feature that allows endpoints to be notified of emerging congestion without packet drops.

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
