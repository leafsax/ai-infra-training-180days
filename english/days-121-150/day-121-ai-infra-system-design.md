### Day 121: Introduction to AI Cluster Network Architecture & RDMA Basics

**1) Topic and Core Examination Areas**
- Topic: Introduction to AI Cluster Network Architecture & RDMA Basics.
- Core Examination Areas: Understanding the role of network architecture in AI training clusters, basic concepts of Remote Direct Memory Access (RDMA), and how RDMA reduces CPU overhead in distributed training.

**2) Requirement Clarification and Metric Definitions**
- **QPS (Queries Per Second)**: In training, not directly applicable, but for distributed sync it relates to synchronization frequency (e.g., 100 syncs/sec for 10-second gradient steps).
- **Latency Metrics (RTT - Round Trip Time)**: Target RTT for intra-node is <1 microsecond (us); for inter-node RDMA networks, target RTT is <1.5 us.
- **Throughput (Bandwidth)**: Target network bandwidth per node for 8-GPU servers is 400 Gbps or 800 Gbps per NIC.
- **Memory Size (HBM - High Bandwidth Memory)**: GPU HBM3 size is typically 80GB-192GB per GPU; network buffers rely on host DRAM and NIC on-board memory.

**3) Core Architecture/Technical Component Design**
- Compute Nodes: GPU servers (e.g., 8x H100) with dual or quad 400G/800G RDMA-capable NICs.
- Network Switches: RDMA switches (InfiniBand or RoCEv2 switches) forming a multi-tier fabric.
- RDMA Protocols: IB (InfiniBand) native or RoCEv2 (RDMA over Converged Ethernet) running on lossless Ethernet.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *RDMA vs Traditional TCP/IP*: Traditional TCP requires CPU intervention for data movement (kernel bypass needed). RDMA allows direct memory-to-memory transfer between nodes without CPU or OS kernel involvement, reducing latency and CPU overhead.
- *RDMA Operations*: Read, Write, Send/Receive, Atomic operations. In AI training, Send/Receive and Read/Write are used for All-Reduce and parameter sync.

**5) Trade-off analysis**
- *RDMA (InfiniBand/RoCEv2) vs TCP*: RDMA offers lower latency and higher throughput but requires specialized hardware (IB switches or RoCEv2-capable Ethernet with PFC/ECN). TCP runs on standard Ethernet but has higher CPU overhead and latency.
- *Cost vs Performance*: InfiniBand offers the best out-of-the-box performance for AI but is more expensive and proprietary. RoCEv2 runs on standard Ethernet switches but requires careful configuration for lossless networking.

**6) How to determine the optimal solution**
- For large-scale LLM training (thousands of GPUs), InfiniBand or high-end RoCEv2 (800G) with proper congestion control is optimal. For smaller clusters or cost-sensitive deployments, RoCEv2 on lossless Ethernet or even high-end TCP (with MPATH/TCP BBR) may suffice.

**7) Full names and explanations of all nouns and abbreviations**
- **RDMA**: Remote Direct Memory Access — allows data to be transferred directly from the memory of one computer to that of another without involvement of the operating system or CPU of either computer.
- **RTT**: Round Trip Time — the time it takes for a signal to be sent plus the time it takes for an acknowledgment of that signal to be received.
- **HBM**: High Bandwidth Memory — a type of computer memory that uses 3D stacking to provide higher bandwidth than traditional GDDR memory.
- **NIC**: Network Interface Card — hardware that connects a computer to a computer network.
- **RoCEv2**: RDMA over Converged Ethernet version 2 — carries RDMA traffic over IP networks.
- **InfiniBand (IB)**: A high-performance networking technology primarily used in high-performance computing and enterprise data centers.

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


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Network Reliability & PFC Storms:** For RoCEv2 with DCQCN or InfiniBand, how do you prevent PFC (Priority Flow Control) storms when congestion occurs? What is the exact recovery time and data loss scenario when an 800G NIC or switch link flaps unexpectedly?
- **On Distributed Storage I/O:** For distributed file systems like Lustre/GPFS/Ceph, how do you handle small random I/O patterns from multiple GPU nodes simultaneously? What is the metadata server bottleneck, and how do you scale it without introducing single points of failure?
- **On Gang Scheduling & Preemption:** For Gang Scheduling in K8s/Slurm, what happens if one node in the required gang is unhealthy or evicted? How do you handle preemption fairness across different tenant queues to prevent starvation of long-running training jobs?
