## Day 30: End-to-End LLM Serving System Design (Capstone)

### 1) Topic and Core Examination Areas
- **Topic**: End-to-End LLM Serving System Design.
- **Core Examination Areas**: Integrating all concepts: compute, network, serving engine, batching, KV cache, quantization, routing, and monitoring into a production-ready system.

### 2) Requirement Clarification and Metric Definitions
- **Scenario**: Design a serving system for a 70B parameter LLM, targeting 2,000 QPS, TTFT < 300ms, TP99 latency < 2s, system throughput > 100,000 tokens/sec.
- **Hardware**: NVIDIA H100 80GB GPUs, InfiniBand network, Kubernetes orchestration.

### 3) Core Architecture/Technical Component Design
- **Frontend**: API Gateway with rate limiting, authentication, and latency-based routing to serving replicas.
- **Serving Layer**: vLLM or TensorRT-LLM instances with PagedAttention, continuous batching, and INT4 quantization (if accuracy allows).
- **Infrastructure**: Kubernetes with NVIDIA GPU Operator, Prometheus/Grafana for monitoring, distributed storage for model weights.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Hybrid Parallelism for Serving**: If the 70B model requires TP=4 or TP=8 per replica, ensure NVLink or high-speed InfiniBand is utilized for intra-replica communication.
- **Auto-scaling**: K8s HPA (Horizontal Pod Autoscaler) or custom controller scaling vLLM pods based on queue length or GPU utilization.

### 5) Trade-off analysis
- **vLLM vs. TensorRT-LLM for 70B**: vLLM offers easier deployment and dynamic batching but TensorRT-LLM may offer higher throughput and lower latency after optimization and conversion. Quantization (INT4) reduces HBM usage and increases batch size but requires kernel support and accuracy validation.

### 6) How to determine the optimal solution
- Start with vLLM for rapid deployment and validation of SLAs (TTFT, throughput). If production requires maximum throughput and latency optimization, convert the model to TensorRT-LLM format, apply INT4 quantization (AWQ/GPTQ), and deploy with TP=4/8 on H100 80GB nodes with InfiniBand. Implement auto-scaling and comprehensive monitoring.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **End-to-End Serving System**: The complete infrastructure stack from API gateway to model execution and monitoring, designed to serve LLMs to end users.
- **HPA (Horizontal Pod Autoscaler)**: A Kubernetes feature that automatically scales the number of pod replicas based on observed metrics like CPU or custom metrics (e.g., queue length).
- **INT4 Quantization**: Reducing model weights to 4-bit integer precision to save memory and accelerate inference, as discussed in Day 12.

--- 

*End of AI Infra System Design Training Question Set: Days 1-30.*