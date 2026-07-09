## Day 19: Monitoring, Profiling, and Observability in AI Infra

### 1) Topic and Core Examination Areas
- **Topic**: Monitoring, Profiling, and Observability.
- **Core Examination Areas**: Metrics (GPU utilization, memory, latency), tracing, logging, and profiling tools (Nsight, PyTorch Profiler).

### 2) Requirement Clarification and Metric Definitions
- **Metrics Collection Interval**: e.g., 1-second intervals for GPU utilization and memory.
- **Alerting Thresholds**: e.g., alert if GPU utilization < 50% for 5 minutes (underutilization) or HBM usage > 95% (OOM risk).

### 3) Core Architecture/Technical Component Design
- **Metrics Pipeline**: Prometheus (metrics collection) + Grafana (visualization) + Alertmanager (alerting).
- **Profiling Tools**: NVIDIA Nsight Systems, PyTorch Profiler, vLLM built-in metrics.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **GPU Metrics**: SM (Streaming Multiprocessor) utilization, memory bandwidth utilization, PCIe/NVLink traffic.
- **Application Tracing**: OpenTelemetry for tracing request flow through API gateway, scheduler, and kernel executor.

### 5) Trade-off analysis
- **Overhead vs. Visibility**: High-frequency metrics and detailed tracing provide excellent visibility but can introduce performance overhead and storage costs. Selective profiling and sampling can balance this.

### 6) How to determine the optimal solution
- Implement a standard observability stack (Prometheus + Grafana) for GPU and serving metrics. Use Nsight Systems or PyTorch Profiler for periodic deep-dive profiling to identify bottlenecks (e.g., kernel launch overhead, communication bottlenecks).

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Observability**: The ability to understand the internal state of a system by examining its outputs (metrics, logs, traces).
- **Prometheus**: An open-source monitoring and alerting toolkit designed for reliability and scalability.
- **Grafana**: An open-source platform for monitoring and observability, often used with Prometheus for visualization.
- **Nsight Systems**: NVIDIA's performance analysis tool for profiling CUDA applications and GPU utilization.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Distributed Training)**:
  ```mermaid
  graph TD
      A[Data Loader] --> B[Training Nodes GPU+CPU]
      B --> C[NCCL All-Reduce Network]
      B --> D[Checkpoint Storage S3/NFS]
      B --> E[Model Registry MLflow]
      C --> B
  ```

- **Data Flow Diagram (Training)**:
  ```mermaid
  flowchart LR
      A[Training Data] --> B[Gradient Compute per GPU]
      B --> C[NCCL Gradient Sync]
      C --> D[Weight Update]
      D --> E[Checkpoint Save]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Inference Batching & Latency:** You mentioned Static vs. Continuous Batching. What is the exact kernel fusion overhead when dynamically splitting batches mid-generation? How do you guarantee TP99 latency when a long-context request (e.g., 128K tokens) arrives and forces KV cache eviction for shorter requests?
- **On Distributed Training Communication:** In DP/TP/PP, how do you handle straggler nodes in a 1024-GPU cluster? If one GPU is 5% slower, does the entire All-Reduce step block? What is the exact communication volume per step for NCCL All-Reduce with BF16 weights and gradients?
- **On Memory Optimization:** You cited ZeRO and PagedAttention. For ZeRO-3, how do you minimize the cross-node communication overhead during the optimizer step? For PagedAttention, what is the exact eviction policy (LRU vs. LFU vs. Attention-score-based) when HBM fragmentation occurs?
