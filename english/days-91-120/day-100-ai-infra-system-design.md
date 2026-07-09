### Day 100: Checkpointing Deep Dive: Distributed Checkpointing (e.g., FSDP, DeepSpeed)

**1) Topic and Core Examination Areas**
- Topic: Checkpointing Deep Dive: Distributed Checkpointing (FSDP, DeepSpeed ZeRO).
- Core Examination Areas: How distributed training frameworks handle checkpointing across multiple GPUs and nodes.

**2) Requirement Clarification and Metric Definitions**
- **DP (Data Parallelism)**: Replicating the model across multiple GPUs, each processing a different batch of data.
- **TP (Tensor Parallelism)**: Splitting model tensors across multiple GPUs.
- **PP (Pipeline Parallelism)**: Splitting model layers across multiple GPUs or nodes.
- **FSDP (Fully Sharded Data Parallel)**: PyTorch's technique for sharding model states across data parallel ranks.

**3) Core Architecture/Technical Component Design**
- **Distributed Checkpointing Workflow**: Each GPU/worker saves its sharded state to a shared storage system, or a rank-0 worker gathers all states and saves a unified checkpoint.
- **FSDP Checkpointing**: Uses `sharded_state_dict` to save only the sharded portions, reducing per-GPU memory and I/O during save.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **DeepSpeed ZeRO Checkpointing**: ZeRO-1/2/3 have different checkpointing behaviors. ZeRO-3 shards parameters, optimizer states, and gradients, requiring all ranks to participate in checkpoint save/restore.
- **FSDP Checkpointing**: PyTorch FSDP provides `save_state_dict` and `load_state_dict` with sharding support, compatible with distributed file systems.

**5) Trade-off analysis**
- *Unified vs. Sharded Checkpoints*: Unified checkpoints (saved by rank-0) are easier to manage and restore but require rank-0 to handle large I/O. Sharded checkpoints (each rank saves its part) distribute I/O but require coordination during restore.
- *ZeRO-2 vs. ZeRO-3 Checkpointing*: ZeRO-3 reduces memory further but increases communication and checkpointing complexity compared to ZeRO-2.

**6) How to determine the optimal solution**
- Use FSDP or DeepSpeed ZeRO-3 for models that exceed GPU memory. Ensure the distributed checkpointing strategy uses sharded saves to all ranks in parallel, writing to a high-performance distributed file system or S3-compatible object storage with Multipart Upload support.

**7) Full names and explanations of all nouns and abbreviations**
- **FSDP (Fully Sharded Data Parallel)**: A PyTorch distributed training technique that shards model states (parameters, gradients, optimizer states) across data parallel ranks to reduce memory usage.
- **ZeRO-1/2/3**: Stages of DeepSpeed's Zero Redundancy Optimizer, progressively sharding optimizer states, gradients, and parameters to reduce memory footprint.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Checkpointing)**:
  ```mermaid
  graph TD
      A[Training Cluster GPUs] --> B[Checkpoint Manager]
      B --> C[Shared Storage S3/NFS/Ceph]
      A --> D[Model Registry MLflow/W&B]
      A --> E[Gradient Sync RDMA/NCCL]
  ```

- **Data Flow Diagram (Checkpoint Flow)**:
  ```mermaid
  flowchart LR
      A[Training Step Compute] --> B[State Snapshot Weights/Optimizer]
      B --> C[Async Checkpoint Writer]
      C --> D[Durable Storage]
      D --> E[Model Registry Update]
  ```
