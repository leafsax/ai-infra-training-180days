## Day 48: Serving System Architecture - vLLM, TGI, TensorRT-LLM Comparison

### 1) Topic and Core Examination Areas
**Topic**: LLM Serving Frameworks Comparison.
**Core Examination Areas**: vLLM, Hugging Face TGI (Text Generation Inference), and NVIDIA TensorRT-LLM.

### 2) Requirement Clarification and Metric Definitions
- **Framework Selection Criteria**: Throughput, latency, ease of use, model support, and quantization support.
- **TPS Target**: e.g., 5000 tokens/sec per 8xH100 node.

### 3) Core Architecture/Technical Component Design
- **vLLM**: Uses PagedAttention and continuous batching. Python-heavy, easy to integrate with Hugging Face models.
- **TGI**: Hugging Face's production serving framework, supports quantization (GPTQ, AWQ) and speculative decoding.
- **TensorRT-LLM**: NVIDIA's high-performance framework using kernel fusion and TensorRT optimizations for maximum throughput.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **vLLM**: Best balance of ease-of-use and high throughput via PagedAttention. Strong community support.
- **TensorRT-LLM**: Highest performance for NVIDIA GPUs, but requires model conversion to TensorRT format and is less flexible for rapid model experimentation.
- **TGI**: Good for Hugging Face ecosystem integration, supports many quantization formats and serving features out-of-the-box.

### 5) Trade-off Analysis
- **vLLM**: High throughput, easy to use, but less low-level optimization than TensorRT-LLM.
- **TensorRT-LLM**: Maximum performance, but steeper learning curve and requires NVIDIA ecosystem.
- **TGI**: Excellent Hugging Face integration, but throughput may not match vLLM or TensorRT-LLM for very large batches.

### 6) How to Determine the Optimal Solution
For rapid deployment and ease of use with Hugging Face models, vLLM or TGI. For maximum production throughput on NVIDIA GPUs with converted models, TensorRT-LLM.

### 7) Glossary: Full Names and Explanations
- **vLLM**: An open-source LLM serving framework that uses PagedAttention and continuous batching for high-throughput inference.
- **TGI (Text Generation Inference)**: Hugging Face's production-ready framework for serving LLMs, supporting quantization and speculative decoding.
- **TensorRT-LLM**: NVIDIA's high-performance library for optimizing and serving LLMs using TensorRT kernel fusion and optimizations.

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
