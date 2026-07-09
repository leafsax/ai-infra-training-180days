### Day 91: Introduction to Checkpointing in Distributed Training & Fault Tolerance Basics

**1) Topic and Core Examination Areas**
- Topic: Introduction to Checkpointing in Distributed Training and Fault Tolerance Basics.
- Core Examination Areas: Purpose of checkpointing in large-scale AI training, fault tolerance mechanisms, checkpoint frequency, and basic recovery workflows.

**2) Requirement Clarification and Metric Definitions**
- **QPS (Queries Per Second)**: Not directly applicable to training, but inference QPS post-training is typically 50-200 QPS for a 7B model.
- **TTFT (Time to First Token)**: Latency metric for inference, target < 200ms for interactive chat.
- **TP99 Latency**: 99th percentile latency for checkpoint save/restore operations. Target: Save TP99 < 30 seconds for a 70B model checkpoint.
- **HBM (High Bandwidth Memory)**: GPU memory size. For a 70B model, FP16/BF16 requires ~140GB per replica; with optimizer states (ZeRO-1), ~350GB per GPU (e.g., 8x H100 80GB).

**3) Core Architecture/Technical Component Design**
- **Training Cluster Architecture**: Compute nodes (GPU servers), storage nodes (distributed storage like Ceph or NFS), and control plane (Kubernetes or Slurm).
- **Checkpointing Workflow**: Periodic snapshot of model weights, optimizer states, and training step metadata to persistent storage.
- **Recovery Workflow**: On node failure, the control plane provisions a new node, restores the latest checkpoint from storage, and resumes training.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Periodic Checkpointing**: Save checkpoint every N steps (e.g., every 1000 steps). Simple but risks data loss up to N steps.
- **Event-Driven Checkpointing**: Save on specific events (e.g., epoch end, validation metric improvement).
- **Continuous Checkpointing**: Incrementally save state changes to reduce recovery time.

**5) Trade-off analysis**
- *Frequency vs. Overhead*: More frequent checkpoints reduce potential data loss but increase I/O overhead and slow down training throughput.
- *Storage Cost vs. Recovery Time*: Storing more historical checkpoints increases storage cost but provides more recovery points.

**6) How to determine the optimal solution**
- Calculate the checkpointing overhead percentage: (Checkpoint save time / Training time per step) * 100. Target < 5% overhead.
- Determine acceptable data loss window based on business requirements (e.g., maximum 1 hour of training data loss).

**7) Full names and explanations of all nouns and abbreviations**
- **Checkpointing**: The process of saving the state of a training job (weights, optimizer states, step number) to persistent storage.
- **Fault Tolerance**: The ability of a system to continue operating properly in the event of the failure of some of its components.
- **HBM (High Bandwidth Memory)**: A type of computer memory that uses 3D stacking to provide higher bandwidth than traditional GDDR memory.
- **TP99**: 99th percentile latency, meaning 99% of operations complete within this time.

---

