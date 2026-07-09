### Day 143: GPU Scheduling Strategies: Time-slicing, MIG, Preemption

**1) Topic and Core Examination Areas**
- Topic: GPU Scheduling Strategies: Time-slicing, MIG, Preemption.
- Core Examination Areas: Understanding how to share GPUs among multiple jobs or users, including time-slicing, NVIDIA MIG (Multi-Instance GPU), and job preemption strategies.

**2) Requirement Clarification and Metric Definitions**
- **Time-Slice Interval**: For GPU time-slicing, the interval between context switches should be 10-100 milliseconds to minimize overhead while providing fair access.
- **MIG Partition Sizes**: NVIDIA MIG allows partitioning a GPU into up to 7 instances (e.g., 1g.5gb, 2g.10gb, 3g.20gb, etc.), each with dedicated memory and compute units.
- **Preemption Latency**: Time to suspend a lower-priority job and free GPUs for a higher-priority job. Target <30 seconds including checkpoint save or context switch.

**3) Core Architecture/Technical Component Design**
- *Time-Slicing Scheduler*: The Kubernetes GPU scheduler can allocate a GPU to multiple Pods and time-share it using round-robin or fair-share algorithms.
- *MIG (Multi-Instance GPU)*: A hardware-level partitioning feature in NVIDIA A100/H100 GPUs that creates isolated GPU instances with dedicated memory, cache, and compute units.
- *Preemption Mechanism*: The scheduler can suspend or evict lower-priority Pods to make resources available for higher-priority jobs.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Time-Slicing vs MIG*: Time-slicing is a software-level sharing mechanism that context-switches between Pods on the same GPU, incurring some overhead. MIG is hardware-level isolation, offering true performance guarantees but reducing the total number of available GPUs (each MIG instance is a separate schedulable unit).
- *Preemption with Checkpointing*: For training jobs, preemption often requires saving a checkpoint before eviction. For inference or stateless jobs, preemption can be instantaneous (container restart).

**5) Trade-off analysis**
- *Time-Slicing vs Dedicated GPUs*: Time-slicing improves GPU utilization for small or bursty workloads but can cause performance degradation due to context switching and memory contention. Dedicated GPUs ensure performance but may be underutilized.
- *MIG vs Full GPU*: MIG allows fine-grained sharing but requires workload compatibility with MIG partition sizes. Full GPU allocation is simpler but less flexible for multi-tenant environments.

**6) How to determine the optimal solution**
- For multi-tenant clusters with diverse workload sizes, use MIG for workloads that fit MIG partition profiles and require performance isolation. Use time-slicing for small, interactive, or inference workloads where slight overhead is acceptable. Implement preemption with automatic checkpointing for training jobs.

**7) Full names and explanations of all nouns and abbreviations**
- **MIG**: Multi-Instance GPU — a feature in NVIDIA A100/H100 GPUs that allows a single GPU to be partitioned into multiple independent GPU instances.
- **Context Switch**: The process of saving the state of a running process or thread and restoring the state of another, allowing multiple processes to share a single CPU or GPU.
- **Preemption**: The act of suspending or evicting a lower-priority job to allocate resources to a higher-priority job.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Task Scheduling)**:
  ```mermaid
  graph TD
      A[User Job Submission] --> B[Scheduler K8s/Slurm]
      B --> C[Gang Scheduling Controller]
      C --> D[Resource Affinity Checker]
      D --> E[GPU Node Allocation]
      E --> F[Job Execution]
  ```

- **Data Flow Diagram (Scheduling Flow)**:
  ```mermaid
  flowchart LR
      A[Job Request] --> B[Queue & Priority Sort]
      B --> C[Resource Availability Check]
      C --> D[Allocate Gang of Pods]
      D --> E[Start Training Job]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Network Reliability & PFC Storms:** For RoCEv2 with DCQCN or InfiniBand, how do you prevent PFC (Priority Flow Control) storms when congestion occurs? What is the exact recovery time and data loss scenario when an 800G NIC or switch link flaps unexpectedly?
- **On Distributed Storage I/O:** For distributed file systems like Lustre/GPFS/Ceph, how do you handle small random I/O patterns from multiple GPU nodes simultaneously? What is the metadata server bottleneck, and how do you scale it without introducing single points of failure?
- **On Gang Scheduling & Preemption:** For Gang Scheduling in K8s/Slurm, what happens if one node in the required gang is unhealthy or evicted? How do you handle preemption fairness across different tenant queues to prevent starvation of long-running training jobs?
