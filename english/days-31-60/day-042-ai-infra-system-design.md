## Day 42: Model Quantization - INT8, INT4, AWQ, GPTQ for Inference Acceleration

### 1) Topic and Core Examination Areas
**Topic**: Model Quantization Techniques for LLM Inference.
**Core Examination Areas**: Precision reduction (FP16/BF16 to INT8/INT4), per-channel vs per-tensor quantization, and algorithms like AWQ and GPTQ.

### 2) Requirement Clarification and Metric Definitions
- **FP16/BF16 (Half Precision)**: 16-bit floating point, standard for LLM inference/training. 70B model = 140GB.
- **INT8 (8-bit Integer)**: Reduces model size by 2x, 70B model = 70GB.
- **INT4 (4-bit Integer)**: Reduces model size by 4x, 70B model = 35GB.
- **Quantization Error**: The accuracy degradation caused by reducing numerical precision.

### 3) Core Architecture/Technical Component Design
- **Quantization Engine**: Converts model weights from FP16 to INT8/INT4 and scales them dynamically during inference.
- **Dequantization Layer**: Converts INT weights back to FP for computation, or uses INT-optimized CUDA kernels (e.g., INT4 matmul).

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Post-Training Quantization (PTQ)**: Quantizes a pre-trained model without further training. 
  - *AWQ (Activation-aware Weight Quantization)*: Preserves important weights based on activation distributions.
  - *GPTQ (Generative Pre-trained Transformer Quantization)*: Uses second-order derivative information to quantize weights with minimal accuracy loss.
- **Quantization-Aware Training (QAT)**: Simulates quantization during training to optimize weights for low precision.

### 5) Trade-off Analysis
- **FP16/BF16**: Highest accuracy, but high memory and compute cost.
- **INT8/INT4 PTQ (AWQ/GPTQ)**: Significant memory and throughput gains, with minimal accuracy loss if done correctly (e.g., GPTQ for 4-bit).

### 6) How to Determine the Optimal Solution
For production LLM serving where memory and throughput are critical, 4-bit quantization using GPTQ or AWQ is optimal, provided the accuracy drop is acceptable for the use case.

### 7) Glossary: Full Names and Explanations
- **Quantization**: The process of converting model weights and activations from high-precision (e.g., FP16, FP32) to low-precision (e.g., INT8, INT4) formats to reduce memory and accelerate computation.
- **FP16/BF16**: 16-bit floating-point formats. FP16 is standard half-precision; BF16 (Brain Floating Point) has the same dynamic range as FP32 but 16 bits, avoiding underflow during training.
- **INT8/INT4**: 8-bit and 4-bit integer formats used for quantized model weights and activations.
- **AWQ (Activation-aware Weight Quantization)**: A quantization method that identifies and preserves important weights based on the distribution of activations.
- **GPTQ (Generative Pre-trained Transformer Quantization)**: A post-training quantization algorithm that uses second-order information to minimize accuracy loss when quantizing LLMs to low precision (e.g., 4-bit).

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
