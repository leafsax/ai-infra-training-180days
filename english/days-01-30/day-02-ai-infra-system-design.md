## Day 2: Compute Architecture for AI (GPU/TPU/NPU)

### 1) Topic and Core Examination Areas
- **Topic**: Compute Architecture for AI Accelerators.
- **Core Examination Areas**: GPU/TPU/NPU architectures, CUDA cores, Tensor Cores, memory bandwidth, and compute throughput (TFLOPS).

### 2) Requirement Clarification and Metric Definitions
- **Compute Throughput**: Target > 1,000 TFLOPS (FP16) per GPU node (e.g., NVIDIA H100).
- **Memory Bandwidth**: Target > 3 TB/s for HBM3 memory.
- **Latency Metrics**: Kernel launch latency < 10 microseconds.
- **QPS/TPS**: Dependent on compute throughput and model size.

### 3) Core Architecture/Technical Component Design
- **GPU Architecture**: Streaming Multiprocessors (SMs), Tensor Cores, L1/L2 caches, HBM memory interface.
- **Inter-GPU Communication**: NVLink (high-speed GPU-to-GPU interconnect) vs. PCIe (host-to-GPU and GPU-to-GPU via motherboard).

### 4) Deep Dive into Key Technologies and Possible Solutions
- **CUDA vs. ROCm**: NVIDIA's CUDA platform vs. AMD's ROCm platform for GPU programming.
- **Tensor Cores**: Specialized hardware units for mixed-precision matrix multiplications, critical for LLM inference and training.
- Comparative analysis: CUDA has a mature ecosystem and wider framework support; ROCm is open-source but has a smaller ecosystem.

### 5) Trade-off analysis
- **Ecosystem Maturity vs. Cost/Openness**: NVIDIA GPUs offer the best software ecosystem (CUDA, cuBLAS, cuDNN) but at a premium price. AMD/TPU offer cost advantages or specific optimizations but may require porting efforts.

### 6) How to determine the optimal solution
- For production LLM serving with maximum framework compatibility and optimized kernels, NVIDIA GPUs (H100, A100) with CUDA are the default choice. Consider TPU or AMD GPUs if cost optimization or specific cloud provider incentives apply.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **GPU (Graphics Processing Unit)**: A parallel processing unit originally designed for graphics, now widely used for AI compute.
- **TPU (Tensor Processing Unit)**: Google's custom ASIC designed specifically for machine learning workloads.
- **NPU (Neural Processing Unit)**: An ASIC or specialized processor designed for neural network and AI workloads.
- **TFLOPS (TeraFLOPS)**: Trillion floating-point operations per second, a measure of compute throughput.
- **CUDA (Compute Unified Device Architecture)**: NVIDIA's parallel computing platform and programming model.
- **ROCm (Radeon Open Compute)**: AMD's open-source software platform for GPU computing.
- **Tensor Cores**: Specialized compute units on modern GPUs for fast matrix multiplication and accumulation.
- **NVLink**: A high-speed, direct GPU-to-GPU interconnect technology developed by NVIDIA.
- **PCIe (Peripheral Component Interconnect Express)**: A high-speed serial computer expansion bus standard.

---

