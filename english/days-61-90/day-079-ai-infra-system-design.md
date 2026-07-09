### **Day 79: Model Serving Auto-Scaling (vLLM, TensorRT-LLM, KServe, TorchServe)**

**1) Topic and Core Examination Areas**
- Model serving frameworks and their scaling characteristics
- Continuous batching and request queue management
- Multi-model serving and tenant isolation

**2) Requirement Clarification and Metric Definitions**
- Model serving QPS: 5,000 QPS per model endpoint
- GPU memory (HBM) per A100: 80 GB
- Throughput: 1,000 tokens/second/gpu
- P99 latency: < 2 seconds for end-to-end generation

**3) Core Architecture/Technical Component Design**
- Serving Framework: vLLM (PagedAttention, continuous batching), TensorRT-LLM (optimized CUDA kernels), KServe (Kubernetes-native serving)
- Queue Management: Request queue with timeout and priority
- Scaling: Pod-level scaling of serving instances

**4) Deep Dive into Key Technologies and Possible Solutions**
- **vLLM vs TensorRT-LLM**: vLLM is framework-agnostic, supports continuous batching via PagedAttention, and is easy to deploy. TensorRT-LLM is NVIDIA's optimized framework, offering higher throughput but requiring model conversion and is GPU-specific.
- **Static Batching vs Continuous Batching**: Static batching groups requests into fixed-size batches and processes them together; continuous batching (or request-level batching) dynamically adds new requests to a batch as older requests finish, maximizing GPU utilization.

**5) Trade-off analysis**
- vLLM: Flexible, open-source, supports many models; but may have lower peak throughput than compiled frameworks.
- TensorRT-LLM: Highest performance for NVIDIA GPUs; but requires conversion, is less flexible, and ties to NVIDIA ecosystem.
- Static vs Continuous Batching: Static is simpler but wastes GPU cycles when batches are incomplete; continuous batching maximizes throughput but is complex to implement.

**6) How to determine the optimal solution**
- For flexibility and ease of deployment with many models, choose vLLM.
- For maximum throughput on NVIDIA GPUs with a specific model, choose TensorRT-LLM.
- Ensure continuous batching is enabled for high-QPS workloads to maximize HBM and compute utilization.

**7) Full names and explanations of nouns and abbreviations**
- **vLLM**: A fast and easy-to-use library for LLM inference and serving.
- **TensorRT-LLM**: NVIDIA's library for high-performance LLM inference on GPUs.
- **KServe**: Kubernetes-native serving framework for machine learning models.
- **PagedAttention**: A memory management technique used by vLLM to efficiently manage KV Cache by paginating it like an OS.
- **HBM**: High Bandwidth Memory. A type of computer memory used in GPUs.
- **KV Cache**: Key-Value Cache. The cached attention keys and values from previous tokens to avoid recomputation.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Auto-Scaling)**:
  ```mermaid
  graph TD
      A[API Gateway] --> B[Inference Service vLLM]
      B --> C[Custom Metrics Server]
      C --> D[KEDA Scaler]
      D --> E[Kubernetes Cluster]
      E --> B
  ```

- **Data Flow Diagram (Auto-Scaling Flow)**:
  ```mermaid
  flowchart LR
      A[Request Load] --> B[Metrics Exporter]
      B --> C[Prometheus / DCGM]
      C --> D[KEDA Operator]
      D --> E[Scale In/Out Pods]
  ```
