### Day 122: InfiniBand vs RoCEv2 vs TCP in AI Clusters

**1) Topic and Core Examination Areas**
- Topic: InfiniBand vs RoCEv2 vs TCP in AI Clusters.
- Core Examination Areas: Comparative analysis of IB, RoCEv2, and TCP for AI workloads, focusing on latency, throughput, ecosystem, and configuration complexity.

**2) Requirement Clarification and Metric Definitions**
- **Latency (Tail Latency TP99)**: Target TP99 latency for collective operations (e.g., All-Reduce) should be <100 microseconds at scale.
- **Throughput (TPS/Bandwidth)**: Target line-rate bandwidth of 400 Gbps or 800 Gbps per link. Packet loss rate should be <10^-5 for RoCEv2/IB to avoid TCP/RDMA retransmissions.
- **QPS/Sync Rate**: Gradient synchronization frequency, e.g., 50-200 syncs per second depending on model size and batch size.

**3) Core Architecture/Technical Component Design**
- InfiniBand Architecture: Dedicated IB switches (e.g., NVIDIA Quantum), IB HCAs (Host Channel Adapters), and IB cables.
- RoCEv2 Architecture: RDMA-capable Ethernet NICs (e.g., Mellanox/NVIDIA ConnectX), top-of-rack (ToR) switches supporting PFC (Priority Flow Control) and ECN (Explicit Congestion Notification).
- TCP Architecture: Standard Ethernet NICs, IP routing, TCP stack optimizations (e.g., RDMA over TCP alternatives like Soft-RoCE or MP-RDMA).

**4) Deep Dive into Key Technologies and Possible Solutions**
- *InfiniBand*: Proprietary network protocol and hardware stack. Offers zero-copy, kernel bypass, and hardware-managed congestion control.
- *RoCEv2*: Runs RDMA over UDP/IP. Requires a lossless Ethernet network configured with PFC (to prevent buffer overflow) and ECN (to signal congestion).
- *TCP with AI optimizations*: Uses standard Ethernet but relies on software optimizations like MPATH (multipath TCP), BBR congestion control, and kernel bypass libraries (e.g., SPDK, AF_XDP).

**5) Trade-off analysis**
- *InfiniBand*: Highest performance, lowest latency, but high cost and vendor lock-in (primarily NVIDIA/Mellanox).
- *RoCEv2*: Lower hardware cost (uses Ethernet), but requires expert tuning for lossless networking (PFC deadlocks, ECN tuning).
- *TCP*: Lowest hardware cost, universally compatible, but higher CPU overhead and latency, not ideal for ultra-large scale LLM training without significant software optimization.

**6) How to determine the optimal solution**
- For >1,000 GPUs training LLMs, InfiniBand or carefully tuned RoCEv2 (800G) is required. For <500 GPUs or fine-tuning workloads, RoCEv2 or even high-performance TCP may be sufficient.

**7) Full names and explanations of all nouns and abbreviations**
- **IB**: InfiniBand — high-throughput, low-latency network protocol.
- **RoCEv2**: RDMA over Converged Ethernet version 2.
- **PFC**: Priority Flow Control — a link-level flow control mechanism in Ethernet to pause specific traffic classes to prevent packet loss.
- **ECN**: Explicit Congestion Notification — a mechanism to signal impending network congestion before buffers overflow.
- **HCA**: Host Channel Adapter — the InfiniBand equivalent of a NIC.
- **ToR**: Top-of-Rack — network switches located at the top of a server rack.

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
