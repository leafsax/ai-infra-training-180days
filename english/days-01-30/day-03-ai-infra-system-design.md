## Day 3: Memory Hierarchy and HBM (High Bandwidth Memory)

### 1) Topic and Core Examination Areas
- **Topic**: Memory Hierarchy and High Bandwidth Memory in AI Systems.
- **Core Examination Areas**: GPU memory hierarchy (registers, L1/L2 cache, HBM), memory bandwidth bottlenecks, and memory capacity planning.

### 2) Requirement Clarification and Metric Definitions
- **HBM Capacity**: e.g., 80GB per H100 GPU.
- **Memory Bandwidth**: Target 3-4 TB/s for HBM3.
- **Model Size**: e.g., a 70B parameter model in FP16 requires ~140GB of memory (2 bytes per parameter), necessitating multi-GPU or quantization.

### 3) Core Architecture/Technical Component Design
- **GPU Memory Hierarchy**: Registers (fastest, smallest) -> L1 Cache -> Shared Memory -> L2 Cache -> HBM (largest, high bandwidth).
- **Memory Partitioning**: How model weights, KV Cache, and intermediate activations are distributed across GPU memory.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **HBM vs. GDDR6**: HBM uses a 3D-stacked design with a wide memory bus, offering significantly higher bandwidth but lower capacity per die compared to GDDR6.
- **Memory Compression**: Techniques to reduce the memory footprint of activations or KV Cache.

### 5) Trade-off analysis
- **Bandwidth vs. Capacity**: HBM offers exceptional bandwidth but limited capacity per GPU. GDDR offers higher capacity per chip but lower bandwidth. For LLMs, HBM is essential to avoid memory bandwidth bottlenecks during inference/training.

### 6) How to determine the optimal solution
- For LLM serving and training, select GPUs with the largest HBM capacity and highest bandwidth (e.g., H100 80GB). If model size exceeds single-GPU HBM capacity, employ model parallelism or quantization.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Memory Hierarchy**: The organization of memory storage in a computer system, balancing speed, capacity, and cost (registers -> caches -> RAM -> storage).
- **HBM (High Bandwidth Memory)**: A type of DDR SDRAM with a 3D-stacked design, offering very high bandwidth for AI accelerators.
- **GDDR6 (Graphics Double Data Rate 6)**: A type of memory commonly used in standard GPUs, offering high bandwidth but typically lower than HBM.
- **L1/L2 Cache**: On-chip memory caches used to reduce the latency of accessing frequently used data from HBM.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram**:
  ```mermaid
  graph TD
      A[Client/User] --> B[API Gateway / Load Balancer]
      B --> C[vLLM/TGI Inference Engine]
      C --> D[GPU Cluster H100/A100]
      C --> E[Model Weights Storage NVMe/S3]
      C --> F[KV Cache Manager]
      D --> G[GPU Monitoring DCGM]
  ```

- **Data Flow Diagram**:
  ```mermaid
  flowchart LR
      A[User Prompt] --> B[API Gateway]
      B --> C[Request Queue & Batching]
      C --> D[GPU Inference Compute]
      D --> E[Token Generation]
      E --> F[Response Stream]
  ```
