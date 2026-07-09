### Day 102: Multimodal Data Pipeline: Image, Text, Audio Data Processing

**1) Topic and Core Examination Areas**
- Topic: Multimodal Data Pipeline: Image, Text, Audio Data Processing.
- Core Examination Areas: Data ingestion, preprocessing, tokenization, and batching for multimodal training.

**2) Requirement Clarification and Metric Definitions**
- **Throughput TPS (Tokens Per Second)**: Multimodal training throughput target: 5000-10000 tokens per second per GPU for a 7B VLM.
- **Data Processing Latency**: Time to preprocess an image-text pair. Target < 10ms per pair to avoid becoming a bottleneck for GPU training.
- **Batch Size**: Global batch size for multimodal training: 256-1024 samples (image-text pairs).

**3) Core Architecture/Technical Component Design**
- **Data Ingestion**: Read from object storage (S3) or distributed file system. Use parallel loaders (e.g., PyTorch DataLoader with multiple workers).
- **Preprocessing Pipeline**: Image resizing, normalization, text tokenization using a tokenizer (e.g., Llama tokenizer).
- **Batching**: Group image-text pairs into batches, ensuring padding or dynamic batching to optimize GPU utilization.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Dynamic Batching vs. Static Batching**: Dynamic batching groups samples with similar sequence lengths at runtime. Static batching uses fixed sequence lengths. Dynamic batching improves GPU utilization but adds scheduling overhead.
- **Preprocessing on CPU vs. GPU**: Image preprocessing (resizing, normalization) is typically done on CPU using libraries like Pillow or OpenCV. Tokenization is also CPU-bound.

**5) Trade-off analysis**
- *CPU Preprocessing vs. GPU Preprocessing*: CPU preprocessing is standard and leverages optimized libraries (OpenCV). GPU preprocessing can be faster for large batches but consumes GPU memory and compute that could be used for training.
- *Fixed vs. Dynamic Batching*: Fixed batching is simpler but may lead to padding overhead. Dynamic batching minimizes padding but requires more complex sequence length tracking.

**6) How to determine the optimal solution**
- Use CPU-based preprocessing with PyTorch DataLoader and multiple workers. Use dynamic batching or padding to a maximum sequence length to optimize GPU utilization. Ensure preprocessing throughput matches or exceeds GPU training throughput.

**7) Full names and explanations of all nouns and abbreviations**
- **TPS (Tokens Per Second)**: A metric measuring the number of tokens processed or generated per second during training or inference.
- **PyTorch DataLoader**: A utility in PyTorch that loads data in parallel using multiple worker processes, supporting batching and shuffling.

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


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Checkpointing & Compute Stalls:** For async checkpointing with ZeRO-3 or FSDP, how do you ensure there is no compute stall during the state snapshot phase? What is the exact RPO (Recovery Point Objective) and RTO (Recovery Time Objective) during a sudden node failure or power outage?
- **On Multimodal Alignment:** How do you align the latency of image/audio encoding with text token generation? What happens if the modalities arrive out of sync or if one modality's preprocessing takes 3x longer than the others?
- **On Model Registry & Rollbacks:** In your model registry design, how do you ensure atomic swaps between model versions during a deployment? What is the rollback mechanism if a newly registered model exhibits severe accuracy degradation or safety violations in production?
