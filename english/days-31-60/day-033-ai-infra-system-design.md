## Day 33: LLM Serving - Model Parallelism: Tensor Parallelism (TP) & Pipeline Parallelism (PP)

### 1) Topic and Core Examination Areas
**Topic**: Model Parallelism Strategies for Serving and Training Large Models.
**Core Examination Areas**: Tensor Parallelism (TP), Pipeline Parallelism (PP), and how they split model weights and computations across multiple GPUs.

### 2) Requirement Clarification and Metric Definitions
- **Model Size**: e.g., 70B parameters, requiring multiple GPUs even for inference.
- **GPUs per Node**: Typically 8x H100 GPUs per server node.
- **TP Degree (Tensor Parallelism size)**: Number of GPUs splitting a single layer's weights (e.g., TP=4).
- **PP Degree (Pipeline Parallelism size)**: Number of GPUs splitting model layers sequentially (e.g., PP=2).

### 3) Core Architecture/Technical Component Design
- **Tensor Parallelism Setup**: Splits matrices (e.g., Q, K, V, O matrices in attention) across GPUs. Each GPU computes a portion and then synchronizes via All-Reduce.
- **Pipeline Parallelism Setup**: Divides the transformer layers into stages. Each stage is assigned to a GPU or group of GPUs.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Tensor Parallelism (TP)**: Splits a single layer's computation across multiple GPUs. For attention, QKV projections are split. Requires high-bandwidth interconnect (NVLink) due to frequent All-Reduce operations.
- **Pipeline Parallelism (PP)**: Splits layers across GPUs. Introduces pipeline bubbles (idle time) between stages. Solutions like GPipe (gradient checkpointing across pipeline) or Interleaved 1F1B (One-Forward-One-Backward) minimize bubbles.

### 5) Trade-off Analysis
- **TP**: High communication overhead within the group, but low latency for single-request inference. Best for fitting large models onto available GPUs.
- **PP**: Better for scaling to very large models across many GPUs, but introduces pipeline bubbles and increases latency due to stage synchronization.

### 6) How to Determine the Optimal Solution
For inference on models like Llama-3-70B, TP=4 or TP=8 is typically used with PP=1 to minimize latency. For training, a combination of TP, PP, and DP (Data Parallelism) is used to scale across thousands of GPUs.

### 7) Glossary: Full Names and Explanations
- **Model Parallelism**: Techniques for distributing a single model's weights or computations across multiple GPUs.
- **Tensor Parallelism (TP)**: A model parallelism technique that splits the tensors (matrices) within a layer across multiple GPUs, requiring frequent synchronization via All-Reduce.
- **Pipeline Parallelism (PP)**: A model parallelism technique that partitions the neural network layers into stages, with each stage assigned to a different GPU, processing different micro-batches through the pipeline.
- **All-Reduce**: A collective communication operation that computes the sum (or other reduction) of data across all GPUs and distributes the result to all of them.
- **NVLink**: A high-speed GPU-to-GPU interconnect technology developed by NVIDIA, offering much higher bandwidth than PCIe.

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
