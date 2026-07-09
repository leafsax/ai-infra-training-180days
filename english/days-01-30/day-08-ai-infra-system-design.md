## Day 8: LLM Inference Serving Basics & Architecture

### 1) Topic and Core Examination Areas
- **Topic**: LLM Inference Serving Basics and Architecture.
- **Core Examination Areas**: Request handling, model loading, inference engines, and serving scalability.

### 2) Requirement Clarification and Metric Definitions
- **QPS**: Target 500-2,000 QPS depending on model size and concurrency.
- **TTFT (Time To First Token)**: Target < 300ms for good UX.
- **Inter-Token Latency (ITL)**: Target < 50ms per token for smooth streaming.
- **HBM Usage**: Must fit model weights, KV Cache, and working activations.

### 3) Core Architecture/Technical Component Design
- **Serving Engine Components**: Request manager, scheduler, kernel executor (CUDA kernels), KV Cache manager.
- **Stateless vs. Stateful Serving**: Stateless API frontends routing to stateful serving engines holding model state in GPU memory.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **vLLM vs. HuggingFace TGI vs. TensorRT-LLM**: 
  - *vLLM*: PagedAttention, continuous batching, easy to deploy.
  - *TGI*: HuggingFace ecosystem, good for open-source models, supports quantization.
  - *TensorRT-LLM*: NVIDIA's highly optimized engine, requires model conversion, offers best raw performance.

### 5) Trade-off analysis
- **Ease of Use vs. Raw Performance**: vLLM and TGI are easy to deploy and support dynamic model loading. TensorRT-LLM offers superior performance but requires a complex build and conversion process.

### 6) How to determine the optimal solution
- For rapid deployment and flexibility, use vLLM or TGI. For production environments where maximum throughput and lowest latency are critical, use TensorRT-LLM after model conversion and profiling.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **LLM Inference Serving**: The process of deploying trained LLMs to accept requests, generate responses, and return results to users.
- **Request Manager**: The component in a serving engine that handles incoming API requests, validates them, and places them in a queue.
- **Scheduler**: The component that decides which requests to execute next, often based on batching strategies and latency goals.
- **ITL (Inter-Token Latency)**: The time delay between generating consecutive tokens in a sequence.

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


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Inference Batching & Latency:** You mentioned Static vs. Continuous Batching. What is the exact kernel fusion overhead when dynamically splitting batches mid-generation? How do you guarantee TP99 latency when a long-context request (e.g., 128K tokens) arrives and forces KV cache eviction for shorter requests?
- **On Distributed Training Communication:** In DP/TP/PP, how do you handle straggler nodes in a 1024-GPU cluster? If one GPU is 5% slower, does the entire All-Reduce step block? What is the exact communication volume per step for NCCL All-Reduce with BF16 weights and gradients?
- **On Memory Optimization:** You cited ZeRO and PagedAttention. For ZeRO-3, how do you minimize the cross-node communication overhead during the optimizer step? For PagedAttention, what is the exact eviction policy (LRU vs. LFU vs. Attention-score-based) when HBM fragmentation occurs?
