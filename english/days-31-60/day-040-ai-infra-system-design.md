## Day 40: Storage Systems for AI - High-Speed Storage (NVMe, GPUDirect Storage)

### 1) Topic and Core Examination Areas
**Topic**: AI Storage Infrastructure and Data Loading Optimization.
**Core Examination Areas**: NVMe SSDs, storage throughput for training data, and GPUDirect Storage (GDS).

### 2) Requirement Clarification and Metric Definitions
- **Storage Throughput**: e.g., 10-20 GB/s per node for training data loading.
- **IOPS (Input/Output Operations Per Second)**: Measure of storage device's ability to handle random read/write operations.
- **Checkpoint Size**: e.g., 70B model checkpoint in BF16 is ~140GB, plus optimizer states can exceed 500GB. Checkpoint I/O must be fast to not bottleneck training.

### 3) Core Architecture/Technical Component Design
- **NVMe SSDs**: High-speed storage devices connected via PCIe, offering 5-7 GB/s read speeds per drive.
- **Parallel File Systems**: e.g., Lustre, GPFS, or cloud-native object storage (S3) with high-throughput access.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Standard CPU-mediated I/O**: Data moves from storage -> CPU memory -> GPU memory. Introduces latency and CPU overhead.
- **GPUDirect Storage (GDS)**: Allows NVMe drives to transfer data directly to GPU memory via PCIe, bypassing CPU memory and reducing latency and CPU utilization.

### 5) Trade-off Analysis
- **CPU-mediated I/O**: Simpler to set up, but CPU becomes a bottleneck for data loading at scale.
- **GPUDirect Storage**: Requires compatible GPUs, NVMe drives, and storage drivers, but significantly improves data loading throughput and reduces training idle time.

### 6) How to Determine the Optimal Solution
For large-scale LLM training with high data throughput requirements, GPUDirect Storage combined with high-speed NVMe or parallel file systems is optimal.

### 7) Glossary: Full Names and Explanations
- **NVMe (Non-Volatile Memory Express)**: A high-speed storage interface protocol designed for flash memory (SSDs) connected via PCIe.
- **IOPS (Input/Output Operations Per Second)**: A metric that measures the number of read or write operations a storage device can perform per second.
- **GPUDirect Storage (GDS)**: A technology that enables direct data transfers between storage devices (like NVMe SSDs) and GPU memory, bypassing the host CPU and system RAM.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram**:
  ```mermaid
  graph TD
      A[Load Balancer] --> B[Inference Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU Resource Manager]
      D --> E[Kubernetes / Karpenter]
      B --> F[Telemetry DCGM/OpenTelemetry]
  ```

- **Data Flow Diagram**:
  ```mermaid
  flowchart LR
      A[Incoming Requests] --> B[Continuous Batching Scheduler]
      B --> C[KV Cache Lookup & Update]
      C --> D[GPU Tensor Core Compute]
      D --> E[Output Tokens]
      E --> F[Streaming Response]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On KV Cache & PagedAttention:** You proposed PagedAttention for KV cache management. What happens when HBM is heavily fragmented under varying context lengths? How do you handle cache eviction policies under heavy load without degrading model accuracy or causing hallucinations?
- **On GPU Partitioning (MIG/vGPU):** For MIG or vGPU partitioning, what is the hard isolation guarantee? Can a noisy neighbor in one MIG partition still affect another via shared PCIe lanes or CPU memory bandwidth? How do you measure and enforce SLOs per partition in real-time?
- **On Auto-Scaling & Thrashing:** For KEDA-based auto-scaling, what is the scale-from-zero latency for bringing up a new vLLM pod? How do you prevent scaling thrashing when QPS fluctuates rapidly around the scaling threshold (e.g., +20% then -20% within 10 seconds)?
