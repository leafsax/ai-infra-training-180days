## Day 17: Storage Systems for AI (Checkpointing, Dataset Storage)

### 1) Topic and Core Examination Areas
- **Topic**: Storage Systems for AI Training and Serving.
- **Core Examination Areas**: Checkpoint storage, dataset storage, NVMe SSDs, and distributed file systems.

### 2) Requirement Clarification and Metric Definitions
- **Checkpoint Frequency**: e.g., every 500 steps or every 1 hour.
- **Checkpoint Size**: e.g., 70B model in FP16 with optimizer states can be 300GB-500GB per checkpoint.
- **Storage Throughput**: Target > 10 GB/s for distributed checkpoint saving/loading.

### 3) Core Architecture/Technical Component Design
- **Distributed File System**: e.g., NFS, GPFS, or cloud-native storage (S3, EFS) for storing checkpoints and datasets.
- **Fast Local Storage**: NVMe SSDs on training nodes for temporary staging of checkpoints and datasets.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Async Checkpointing**: Saving checkpoints asynchronously to avoid blocking training steps.
- **Sharded Checkpoints**: Saving model state in shards across multiple storage nodes to maximize write/read bandwidth.

### 5) Trade-off analysis
- **Speed vs. Durability**: Local NVMe storage offers high speed for checkpoint staging but risks data loss if the node fails. Distributed storage (S3, NFS) is durable but may have higher latency and lower throughput unless specifically optimized (e.g., using parallel file systems).

### 6) How to determine the optimal solution
- Use a hybrid approach: Stage checkpoints on local NVMe SSDs and then asynchronously copy them to a durable distributed storage system (S3, NFS) or use framework-native distributed checkpointing (e.g., DeepSpeed, PyTorch FSDP) that shards and parallelizes checkpoint I/O.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Checkpoint**: A saved state of a training process (model weights, optimizer states, step number) used for recovery or evaluation.
- **NVMe (Non-Volatile Memory Express)**: A high-speed interface for SSDs, offering much higher throughput and lower latency than SATA or SAS.
- **FSDP (Fully Sharded Data Parallel)**: A PyTorch technique (similar to ZeRO-3) that shards model states across data parallel workers to reduce memory usage.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Distributed Training)**:
  ```mermaid
  graph TD
      A[Data Loader] --> B[Training Nodes GPU+CPU]
      B --> C[NCCL All-Reduce Network]
      B --> D[Checkpoint Storage S3/NFS]
      B --> E[Model Registry MLflow]
      C --> B
  ```

- **Data Flow Diagram (Training)**:
  ```mermaid
  flowchart LR
      A[Training Data] --> B[Gradient Compute per GPU]
      B --> C[NCCL Gradient Sync]
      C --> D[Weight Update]
      D --> E[Checkpoint Save]
  ```
