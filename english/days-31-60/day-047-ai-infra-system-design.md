## Day 47: AI Inference Optimization - Kernel Fusion and CUDA Optimization Basics

### 1) Topic and Core Examination Areas
**Topic**: Low-Level Inference Optimization.
**Core Examination Areas**: Kernel fusion, CUDA basics, and memory access patterns for GPU acceleration.

### 2) Requirement Clarification and Metric Definitions
- **Kernel**: A function executed on the GPU.
- **Kernel Launch Overhead**: The CPU-to-GPU synchronization cost when starting a new kernel.
- **Memory Coalescing**: Organizing memory accesses so that adjacent threads access adjacent memory addresses, maximizing bandwidth.

### 3) Core Architecture/Technical Component Design
- **Computation Graph**: Represents the model's operations. Fusion combines multiple operations into a single kernel.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Kernel Fusion**: Combines multiple small kernels (e.g., Add + Activation) into one larger kernel to reduce memory read/write operations and kernel launch overhead.
- **CUDA Optimization**: Writing or using optimized CUDA/cuBLAS/cuDNN kernels for specific operations like attention or matmul.

### 5) Trade-off Analysis
- **Unfused Kernels**: Easier to debug and implement, but higher memory traffic and launch overhead.
- **Fused Kernels**: Higher performance, but harder to debug and may increase GPU memory usage for intermediate buffers.

### 6) How to Determine the Optimal Solution
For production inference engines (vLLM, TensorRT-LLM), kernel fusion is standard and essential for achieving maximum throughput and minimum latency.

### 7) Glossary: Full Names and Explanations
- **Kernel (in GPU computing)**: A function that is executed on the GPU, typically in parallel across many threads.
- **Kernel Fusion**: An optimization technique that combines multiple GPU kernels into a single kernel to reduce memory access and launch overhead.
- **CUDA (Compute Unified Device Architecture)**: NVIDIA's parallel computing platform and programming model for utilizing GPUs for general-purpose processing.
- **Memory Coalescing**: A GPU memory access pattern where adjacent threads access adjacent memory locations, maximizing memory bandwidth utilization.

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
