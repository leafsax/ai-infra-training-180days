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
