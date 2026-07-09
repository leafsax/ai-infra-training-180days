### Day 135: Checkpointing Storage: Requirements and Design (Frequent saves, large sizes)

**1) Topic and Core Examination Areas**
- Topic: Checkpointing Storage: Requirements and Design.
- Core Examination Areas: Understanding the storage requirements for AI model checkpoints, including frequency, size, write patterns, and the impact on training throughput.

**2) Requirement Clarification and Metric Definitions**
- **Checkpoint Size**: For a 70B parameter model in FP16, checkpoint size is ~140GB (weights) + optimizer states (~280GB for Adam) = ~420GB per full checkpoint.
- **Checkpoint Frequency**: Every 1000-10000 steps, or every 1-24 hours, depending on cluster stability and job length.
- **Write Throughput**: Saving a 420GB checkpoint should take <60 seconds to avoid stalling training for more than 1-2% of total time. Target checkpoint write throughput >= 7 GB/s.

**3) Core Architecture/Technical Component Design**
- *Asynchronous Checkpointing*: Use background threads or separate storage nodes to write checkpoints asynchronously to avoid blocking GPU training loops.
- *Checkpoint Sharding*: Each GPU or rank saves its local checkpoint (weights, optimizer states) to a shared distributed file system or object storage concurrently.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Distributed File System for Checkpoints*: Lustre or GPFS are preferred for fast, concurrent file writes. Object storage (S3/MinIO) is used for long-term archival but may be too slow for frequent checkpoints without optimization (e.g., multipart uploads, parallel writes).
- *Checkpoint Compression*: Some frameworks compress optimizer states or use quantization during checkpointing to reduce storage size and write time, at the cost of CPU overhead.

**5) Trade-off analysis**
- *Frequent vs Infrequent Checkpoints*: Frequent checkpoints improve recovery time but increase storage I/O load and can stall training. Infrequent checkpoints reduce I/O overhead but increase recovery time in case of failure.
- *Synchronous vs Asynchronous Write*: Synchronous writes ensure data is persisted before resuming training but stall GPUs. Asynchronous writes improve training throughput but risk data loss if the storage node fails before write completion.

**6) How to determine the optimal solution**
- Use a high-performance distributed file system (Lustre/GPFS) or fast object storage (MinIO with NVMe backend) for checkpoints. Implement asynchronous, sharded checkpointing where each GPU/rank writes its local state concurrently. Tune checkpoint frequency based on cluster MTBF and training time per step.

**7) Full names and explanations of all nouns and abbreviations**
- **FP16**: 16-bit Floating Point — a half-precision floating-point format used to reduce memory and bandwidth in AI models.
- **Adam**: Adaptive Moment Estimation — a popular optimization algorithm used in training deep learning models.
- **MTBF**: Mean Time Between Failures — a measure of system reliability.

---

(Continuing to Days 136-150 with the same 7-section structure...)



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
