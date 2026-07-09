## Day 36: LLM Training - Memory Optimization Techniques (Gradient Checkpointing, ZeRO-1/2/3)

### 1) Topic and Core Examination Areas
**Topic**: Advanced Memory Optimization for LLM Training.
**Core Examination Areas**: Gradient Checkpointing (Activation Recomputation), memory profiling, and combining ZeRO with checkpointing.

### 2) Requirement Clarification and Metric Definitions
- **Activations**: Intermediate computations stored for backward pass. Can consume 50-80% of training memory for large batch sizes and long context windows.
- **Batch Size (Micro-batch)**: Number of samples processed before a gradient update.
- **GPU Memory Usage**: Measured in GB; target is to fit model + states + activations within 80GB HBM.

### 3) Core Architecture/Technical Component Design
- **Forward Pass**: Computes activations and stores them.
- **Backward Pass**: Uses stored activations to compute gradients. If not stored, activations must be recomputed (checkpointing).

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Gradient Checkpointing (Activation Recomputation)**: Instead of storing all activations, only a subset (checkpoints) is stored. During the backward pass, missing activations are recomputed in the forward pass. Trades compute for memory.
- **Combined ZeRO + Checkpointing**: ZeRO-3 reduces parameter/optimizer memory, while checkpointing reduces activation memory. This combination enables training of 100B+ models on available GPU hardware.

### 5) Trade-off Analysis
- **Full Activation Storage**: Low compute overhead during backward pass, but high memory usage, limiting batch size or model size.
- **Gradient Checkpointing**: Reduces memory linearly with the checkpointing interval, but increases forward pass compute during backward by 33%-100% depending on strategy.

### 6) How to Determine the Optimal Solution
Use gradient checkpointing with a tuned interval (e.g., checkpoint every 2-4 layers) to balance memory savings and compute overhead. Combine with ZeRO-3/FSDP for maximum model scaling.

### 7) Glossary: Full Names and Explanations
- **Gradient Checkpointing (Activation Recomputation)**: A memory optimization technique where intermediate activations are not stored during the forward pass; instead, they are recomputed during the backward pass to save GPU memory.
- **Activations**: The intermediate output values of neural network layers that must be stored for the backward pass (gradient computation).
- **Micro-batch**: A subset of the training data processed in a single forward/backward pass before gradients are aggregated and applied.

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
