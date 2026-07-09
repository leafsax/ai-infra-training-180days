## Day 39: GPU Interconnects - NVLink, NVSwitch, and GPU-to-GPU Communication

### 1) Topic and Core Examination Areas
**Topic**: GPU Interconnect Technologies.
**Core Examination Areas**: NVLink, NVSwitch, PCIe vs NVLink bandwidth, and their impact on model parallelism and distributed training.

### 2) Requirement Clarification and Metric Definitions
- **PCIe Gen5 Bandwidth**: ~32 GB/s per lane (x16 is ~256 GB/s bidirectional).
- **NVLink Bandwidth**: e.g., H100 NVLink offers 900 GB/s bidirectional per link.
- **NVSwitch**: A switch fabric that connects multiple GPUs within a node or across nodes at NVLink speeds.

### 3) Core Architecture/Technical Component Design
- **GPU-to-GPU Direct Communication**: NVLink enables direct memory access between GPUs without going through the host CPU or PCIe switch.
- **Node Architecture**: 8x H100 GPUs connected via NVSwitch to form a full-mesh NVLink fabric, offering 4.5 TB/s aggregate bandwidth.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **PCIe**: General-purpose CPU-to-GPU and GPU-to-GPU (via host) interconnect. Sufficient for Data Parallelism but a bottleneck for Tensor Parallelism.
- **NVLink + NVSwitch**: Provides a high-bandwidth, low-latency interconnect specifically designed for multi-GPU workloads, enabling efficient Tensor Parallelism and Ring-AllReduce operations.

### 5) Trade-off Analysis
- **PCIe-only**: Lower cost, but bandwidth is insufficient for fine-grained model parallelism (TP>2) at scale.
- **NVLink/NVSwitch**: Essential for high-performance LLM training/serving, but requires specialized hardware (e.g., HGX nodes) and increases cost.

### 6) How to Determine the Optimal Solution
For any LLM training or large-model inference (TP>=4), NVLink/NVSwitch connectivity (HGX nodes) is mandatory. For smaller models or DP-only inference, PCIe may suffice.

### 7) Glossary: Full Names and Explanations
- **NVLink**: A high-speed GPU-to-GPU interconnect technology developed by NVIDIA, offering bandwidths significantly higher than PCIe (e.g., 900 GB/s per link for H100).
- **NVSwitch**: A high-bandwidth switch fabric that connects multiple GPUs within a server node or across nodes, enabling full-mesh NVLink connectivity.
- **PCIe (Peripheral Component Interconnect Express)**: A standard bus interface for connecting hardware devices (like GPUs) to a computer's motherboard and CPU.

---

