## Day 49: AI Training Data Pipeline - Data Loading, Preprocessing, and Distributed Dataloaders

### 1) Topic and Core Examination Areas
**Topic**: Data Pipeline Infrastructure for LLM Training.
**Core Examination Areas**: Distributed dataloaders, data shuffling, preprocessing at scale, and avoiding I/O bottlenecks.

### 2) Requirement Clarification and Metric Definitions
- **Training Dataset Size**: e.g., 10 trillion tokens.
- **Data Loading Throughput**: Must match or exceed GPU compute throughput (e.g., 10-20 GB/s per node).
- **Sharding**: Splitting the dataset across multiple workers/gpus.

### 3) Core Architecture/Technical Component Design
- **Data Ingestion Service**: Pulls data from object storage (S3) or distributed filesystems.
- **Preprocessing Cluster**: Tokenizes and formats data into training sequences.
- **Distributed DataLoader**: Ensures each GPU receives a unique, shuffled subset of the data.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Prefetching and Async Loading**: Overlaps data loading with GPU computation to hide I/O latency.
- **WebDataset / LMFormat**: Efficient data formats for streaming large-scale training data without full local download.

### 5) Trade-off Analysis
- **Full Local Download**: Simple, but wastes storage and bandwidth.
- **Streaming from Object Storage**: Saves local storage, but requires high network bandwidth and efficient streaming clients.

### 6) How to Determine the Optimal Solution
For trillion-token training, streaming data formats (WebDataset, parquet) with async dataloaders and prefetching is optimal to keep GPUs fed without I/O bottlenecks.

### 7) Glossary: Full Names and Explanations
- **Distributed DataLoader**: A component that splits and feeds training data to multiple GPUs or workers in a distributed training setup.
- **Prefetching**: A technique where data is loaded into memory or cache before it is needed by the GPU, hiding I/O latency.
- **WebDataset**: An efficient format and streaming protocol for large-scale machine learning datasets, designed for high-throughput data loading.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (GPU Cluster Management)**:
  ```mermaid
  graph TD
      A[Cluster Controller] --> B[GPU Nodes H100]
      B --> C[MIG / vGPU Partitioner]
      B --> D[RDMA Network Switch]
      A --> E[Auto-Scaler KEDA]
      E --> B
  ```

- **Data Flow Diagram (Scaling)**:
  ```mermaid
  flowchart LR
      A[Traffic Spike] --> B[Metrics Collector DCGM]
      B --> C[Scaler Controller]
      C --> D[Provision New GPU Pods]
      D --> E[Route Traffic to New Pods]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On KV Cache & PagedAttention:** You proposed PagedAttention for KV cache management. What happens when HBM is heavily fragmented under varying context lengths? How do you handle cache eviction policies under heavy load without degrading model accuracy or causing hallucinations?
- **On GPU Partitioning (MIG/vGPU):** For MIG or vGPU partitioning, what is the hard isolation guarantee? Can a noisy neighbor in one MIG partition still affect another via shared PCIe lanes or CPU memory bandwidth? How do you measure and enforce SLOs per partition in real-time?
- **On Auto-Scaling & Thrashing:** For KEDA-based auto-scaling, what is the scale-from-zero latency for bringing up a new vLLM pod? How do you prevent scaling thrashing when QPS fluctuates rapidly around the scaling threshold (e.g., +20% then -20% within 10 seconds)?
