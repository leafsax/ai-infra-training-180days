### Day 146: Fault Tolerance and Checkpoint Recovery in Schedulers

**1) Topic and Core Examination Areas**
- Topic: Fault Tolerance and Checkpoint Recovery in Schedulers.
- Core Examination Areas: Understanding how schedulers and training frameworks handle node failures, Pod evictions, and how to recover jobs using checkpoints.

**2) Requirement Clarification and Metric Definitions**
- **Failure Detection Time**: Time to detect a node or Pod failure. Target <10 seconds for K8s node status updates.
- **Recovery Time**: Time to reschedule a job and restore from the last checkpoint. Target <5 minutes for a 64-GPU job with a 420GB checkpoint.
- **Checkpoint Frequency vs MTBF**: Checkpoint frequency should be tuned so that the expected loss between checkpoints is less than 1-2 hours of training time, based on cluster MTBF.

**3) Core Architecture/Technical Component Design**
- *Pod Restart Policies*: Kubernetes policies (Always, OnFailure, Never) that determine how to handle Pod failures. For training jobs, OnFailure or Always with checkpoint recovery is typical.
- *Scheduler Failure Handling*: AI schedulers (Volcano, etc.) can detect failed ranks and trigger job restart or checkpoint recovery workflows.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Automatic Checkpoint Recovery*: Training frameworks (PyTorch DDP, DeepSpeed, Megatron-LM) can be configured to automatically load the latest checkpoint upon restart, resuming training from the last saved step.
- *Node Drain and Eviction*: When a node is failing or being maintained, the scheduler can drain it (cordon + drain), evicting Pods gracefully and allowing them to restart on healthy nodes with checkpoint recovery.

**5) Trade-off analysis**
- *Frequent Checkpointing vs Recovery Time*: Frequent checkpoints reduce recovery time but increase storage I/O and training overhead. Infrequent checkpoints save I/O but increase recovery time and wasted compute after a failure.
- *Graceful Eviction vs Forceful Termination*: Graceful eviction allows checkpoint saving but delays resource reallocation. Forceful termination is instant but requires full restart from the last checkpoint.

**6) How to determine the optimal solution**
- Configure training frameworks with automatic checkpoint recovery and set checkpoint frequency based on cluster MTBF and storage performance. Use Kubernetes node drain workflows for planned maintenance, and rely on scheduler-driven Pod restarts for unexpected failures.

**7) Full names and explanations of all nouns and abbreviations**
- **DDP**: Distributed Data Parallel — a training paradigm where each GPU processes a subset of the batch and gradients are synchronized across GPUs.
- **DeepSpeed**: Microsoft's library for distributed and memory-efficient training, including ZeRO optimization and checkpointing features.
- **Megatron-LM**: NVIDIA's library for training large transformer models, supporting tensor/pipeline parallelism and checkpointing.

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
