### Day 124: NVLink and NVSwitch: GPU-to-GPU Interconnects

**1) Topic and Core Examination Areas**
- Topic: NVLink and NVSwitch: GPU-to-GPU Interconnects.
- Core Examination Areas: Understanding intra-node GPU communication, NVLink bandwidth, NVSwitch architecture, and how it replaces or complements PCIe for GPU-to-GPU data transfer.

**2) Requirement Clarification and Metric Definitions**
- **Intra-node Bandwidth**: Target NVLink bandwidth for H100 is 900 GB/s (bidirectional) per GPU-to-GPU link. For an 8-GPU server with NVSwitch, aggregate bandwidth is ~4.3 TB/s.
- **Latency**: NVLink latency is <1 microsecond for memory access across GPUs.
- **Memory Size (HBM)**: Each H100 GPU has 80GB or 192GB HBM3; NVLink allows these to be virtually pooled or used for tensor/attention parallelism.

**3) Core Architecture/Technical Component Design**
- NVLink: A direct GPU-to-GPU high-bandwidth interconnect. Each H100 GPU has multiple NVLink ports (e.g., 18 links of 56 GB/s each for bidirectional 900 GB/s).
- NVSwitch: A chip that connects multiple GPUs within a server or across servers, forming a full-mesh intra-node (or intra-subnode) interconnect, eliminating PCIe as the bottleneck for GPU-to-GPU communication.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *NVLink vs PCIe*: PCIe Gen5 x16 provides ~32 GB/s bidirectional bandwidth per slot. NVLink provides 900 GB/s per GPU link pair, making it 20-30x faster for GPU-to-GPU data transfer.
- *NVSwitch Fabric*: In an 8-GPU H100 server, NVSwitch connects all 8 GPUs in a full-mesh, allowing any GPU to communicate with any other GPU at NVLink speeds.

**5) Trade-off analysis**
- *NVLink/NVSwitch vs PCIe*: NVLink/NVSwitch offers vastly superior bandwidth and lower latency but is proprietary to NVIDIA and increases GPU/board cost. PCIe is universal and cheaper but becomes a bottleneck for large batch sizes or model parallelism within a node.
- *Intra-node vs Inter-node*: NVLink handles intra-node (within server) communication; inter-node still relies on RDMA (IB/RoCEv2) or high-speed Ethernet.

**6) How to determine the optimal solution**
- For modern LLM training (DP/TP/PP parallelism), NVLink/NVSwitch is mandatory for intra-node communication. PCIe should only be used for host-GPU data loading or management traffic.

**7) Full names and explanations of all nouns and abbreviations**
- **NVLink**: NVIDIA's high-speed, direct-connected GPU interconnect technology.
- **NVSwitch**: A switch chip designed by NVIDIA to connect multiple GPUs at NVLink speeds, forming a non-blocking fabric.
- **HBM3**: High Bandwidth Memory version 3 — the latest generation of 3D-stacked memory used in modern GPUs.
- **TP**: Tensor Parallelism — splitting a single model layer across multiple GPUs.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Network Architecture)**:
  ```mermaid
  graph TD
      A[Job Scheduler Slurm/K8s] --> B[Compute Nodes GPU+CPU]
      B --> C[RDMA Network InfiniBand/RoCE]
      B --> D[Distributed Storage Lustre/GPFS]
      C --> E[All-Reduce Communication NCCL]
  ```

- **Data Flow Diagram (Training Data & Compute)**:
  ```mermaid
  flowchart LR
      A[Data Loader] --> B[Local NVMe Cache]
      B --> C[GPU Compute Kernel]
      C --> D[NCCL All-Reduce via RDMA]
      D --> C
      C --> E[Gradient Update]
  ```
