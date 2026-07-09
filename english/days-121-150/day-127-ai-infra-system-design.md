### Day 127: Network Switches and Routers in AI Clusters (Spectrum, Tomahawk)

**1) Topic and Core Examination Areas**
- Topic: Network Switches and Routers in AI Clusters.
- Core Examination Areas: Understanding key switch chip families (NVIDIA Spectrum for IB/Ethernet, Tomahawk for Ethernet), port densities, and switching architectures for AI workloads.

**2) Requirement Clarification and Metric Definitions**
- **Switch Port Density**: Target 64 or 128 ports of 400G or 800G per switch.
- **Switching Capacity**: For a 64-port 800G switch, switching capacity should be >= 102.4 Tbps (full duplex).
- **Buffer per Port**: Switch on-chip memory per port should be >= 140 MB to handle RDMA/RoCEv2 micro-bursts without packet loss.

**3) Core Architecture/Technical Component Design**
- *InfiniBand Switches*: NVIDIA Quantum-2 or Spectrum-X IB switches, providing non-blocking IB fabric with hardware congestion control.
- *Ethernet Switches*: NVIDIA Spectrum-4/5 or Broadcom Tomahawk 4/5 switches, supporting RoCEv2, PFC, ECN, and high port densities (64x400G or 32x800G).

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Spectrum vs Tomahawk*: Spectrum series is NVIDIA's switch chip family optimized for both IB and RoCEv2, with tight integration into the NVIDIA AI stack (NCCL, cuNet). Tomahawk is Broadcom's high-density Ethernet switch chip, widely used in general data centers and increasingly for RoCEv2 AI clusters.
- *Time-Sensitive Networking (TSN)*: Some advanced switches support TSN features for deterministic latency, though AI clusters primarily rely on RDMA congestion control rather than TSN.

**5) Trade-off analysis**
- *NVIDIA Spectrum vs Broadcom Tomahawk*: Spectrum offers better out-of-the-box integration with NVIDIA GPUs and NCCL, but Tomahawk switches are often more cost-effective and have a broader third-party ecosystem.
- *IB Switches vs Ethernet Switches*: IB switches are purpose-built for RDMA and AI, offering lower latency and simpler configuration for lossless networks. Ethernet switches are more versatile and cheaper but require careful RoCEv2 tuning.

**6) How to determine the optimal solution**
- For NVIDIA-dominated AI clusters (H100/H200), Spectrum switches (IB or Ethernet) are optimal for seamless NCCL integration. For multi-vendor or cost-optimized clusters, Tomahawk-based RoCEv2 switches with DCQCN are viable.

**7) Full names and explanations of all nouns and abbreviations**
- **NCCL**: NVIDIA Collective Communications Library — a library providing standard communications primitives for distributed GPU training.
- **cuNet**: NVIDIA's network profiling and optimization library for RDMA/Ethernet.
- **TSN**: Time-Sensitive Networking — a set of IEEE standards for deterministic communication over Ethernet.

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
