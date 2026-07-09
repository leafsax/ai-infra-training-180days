## Day 12: Quantization for Inference (INT8, INT4, AWQ, GPTQ)

### 1) Topic and Core Examination Areas
- **Topic**: Quantization Techniques for LLM Inference.
- **Core Examination Areas**: Precision reduction (FP16/BF16 to INT8/INT4), per-channel vs. per-token quantization, and calibration methods.

### 2) Requirement Clarification and Metric Definitions
- **Model Size Reduction**: Quantizing from FP16 (2 bytes/param) to INT4 (0.5 bytes/param) reduces model size by 75%.
- **Accuracy Drop Target**: < 1-2% degradation in benchmark metrics (e.g., MMLU, HELM) after quantization.

### 3) Core Architecture/Technical Component Design
- **Quantization Aware Training (QAT) vs. Post-Training Quantization (PTQ)**: QAT adjusts model weights during training to be quantization-friendly. PTQ applies quantization after training is complete.
- **Quantization Engines**: Support for INT4/INT8 matrix multiplications in GPUs (e.g., NVIDIA Tensor Cores support FP8, INT8; INT4 requires specific kernels).

### 4) Deep Dive into Key Technologies and Possible Solutions
- **AWQ (Activation-aware Weight Quantization)**: Preserves important weights based on activation statistics, allowing safe INT4 quantization with minimal accuracy loss.
- **GPTQ (Generative Pre-trained Transformer Quantization)**: A PTQ method that quantizes weights layer-by-layer using second-order information (Hessian matrix) to minimize error.

### 5) Trade-off analysis
- **FP16/BF16 vs. INT8/INT4**: FP16/BF16 offers maximum accuracy but requires more HBM and bandwidth. INT8/INT4 reduces memory footprint and can increase throughput on supported hardware, but risks accuracy degradation and requires careful calibration or kernels.

### 6) How to determine the optimal solution
- For production serving with strict latency and cost constraints, use INT4 quantization (AWQ or GPTQ) if the serving engine (e.g., vLLM, TensorRT-LLM) supports it and benchmark accuracy loss is acceptable. Otherwise, use FP16/BF16 or INT8.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Quantization**: The process of reducing the numerical precision of model weights and activations (e.g., from FP16 to INT8 or INT4) to save memory and accelerate computation.
- **FP16/BF16 (Float16/Bfloat16)**: 16-bit floating-point formats used in AI compute. BF16 retains the dynamic range of FP32 but with lower precision, useful for training stability.
- **INT8/INT4 (Integer 8-bit/4-bit)**: Low-precision integer formats used for quantized model weights and activations.
- **AWQ (Activation-aware Weight Quantization)**: A quantization method that identifies and preserves important weights based on activation distributions.
- **GPTQ**: A post-training quantization algorithm that uses second-order information to quantize LLM weights with minimal accuracy loss.
- **QAT (Quantization-Aware Training)**: Training a model while simulating quantization effects to improve final quantized accuracy.
- **PTQ (Post-Training Quantization)**: Quantizing a model after training is complete, without further training.

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
