### Day 134: Block Storage and NVMe SSDs for High-Performance AI Training

**1) Topic and Core Examination Areas**
- Topic: Block Storage and NVMe SSDs for High-Performance AI Training.
- Core Examination Areas: Understanding the role of block storage and NVMe SSDs in AI infra, particularly for local GPU node storage, caching, and fast checkpointing.

**2) Requirement Clarification and Metric Definitions**
- **NVMe Throughput**: Target per-NVMe drive sequential read/write speeds of 5-7 GB/s for PCIe Gen4/Gen5 NVMe SSDs.
- **Latency**: NVMe read/write latency should be <100 microseconds.
- **Capacity**: Local node storage for caching or checkpointing should be 1-4 TB per GPU server.

**3) Core Architecture/Technical Component Design**
- *Local NVMe Drives*: Installed directly into GPU servers via PCIe slots, used for local data caching, temporary checkpoints, or OS/application storage.
- *NVMe over Fabrics (NVMe-oF)*: Allows NVMe drives to be accessed over a network (e.g., RoCEv2 or Fibre Channel), providing block-level storage with NVMe performance across the cluster.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Local NVMe vs Network Storage*: Local NVMe offers the lowest latency and highest throughput for node-local operations but is not shared across nodes. NVMe-oF provides shared block storage with NVMe performance but requires a low-latency network (RoCEv2/InfiniBand).
- *NVMe-oF Protocols*: RoCEv2-based NVMe-oF (NVMe/RoCE) and Fibre Channel-based NVMe-oF (NVMe/FC) are the primary options for networked NVMe storage.

**5) Trade-off analysis**
- *Local vs Networked NVMe*: Local NVMe is simpler and faster but lacks centralized management and cross-node sharing. Networked NVMe (NVMe-oF) enables shared fast storage but adds network complexity and requires RDMA-capable networks.
- *Cost vs Performance*: NVMe SSDs are more expensive per GB than HDDs or even SATA SSDs, but are necessary for AI caching and low-latency checkpointing.

**6) How to determine the optimal solution**
- Use local PCIe NVMe drives for node-local caching and temporary storage. For shared fast storage across the cluster, deploy NVMe-oF over RoCEv2 or InfiniBand if ultra-low latency block storage is required for distributed checkpointing.

**7) Full names and explanations of all nouns and abbreviations**
- **NVMe-oF**: NVMe over Fabrics — a set of protocols that allow NVMe commands to be sent over network fabrics (InfiniBand, RoCE, TCP).
- **NVMe/RoCE**: NVMe over RDMA over Converged Ethernet — using RoCEv2 networks to access NVMe storage.
- **Fibre Channel (FC)**: A high-speed network technology primarily used for storage area networks (SANs).

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
