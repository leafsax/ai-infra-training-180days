## Day 52: Monitoring and Observability for AI Infra - Metrics, Logging, Tracing

### 1) Topic and Core Examination Areas
**Topic**: Observability Infrastructure for AI Systems.
**Core Examination Areas**: Metrics collection (GPU utilization, temperature), logging, and distributed tracing for LLM serving and training.

### 2) Requirement Clarification and Metric Definitions
- **GPU Utilization**: Percentage of time the GPU is actively processing data. Target: >80% for training.
- **GPU Memory Usage**: GB of HBM used. Must monitor for OOM (Out of Memory) errors.
- **Metrics Exporters**: e.g., NVIDIA DCGM (Data Center GPU Manager) exporter for Prometheus.

### 3) Core Architecture/Technical Component Design
- **Metrics Collection Pipeline**: Agents on each node collect GPU and system metrics and send to a time-series database (Prometheus).
- **Logging Aggregator**: Collects application and system logs (ELK stack or Loki).
- **Tracing**: OpenTelemetry or Jaeger for tracing LLM serving requests across components.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **DCGM (Data Center GPU Manager)**: NVIDIA's tool for monitoring and managing GPU health, utilization, and metrics.
- **Prometheus + Grafana**: Standard stack for metrics collection and visualization.

### 5) Trade-off Analysis
- **Custom Monitoring Scripts**: Simple, but lack scalability and standard integration.
- **Prometheus/DCGM Stack**: Standardized, scalable, but requires infrastructure setup and maintenance.

### 6) How to Determine the Optimal Solution
For production AI infra, the Prometheus + DCGM exporter + Grafana stack for metrics, combined with centralized logging (Loki/ELK), is optimal.

### 7) Glossary: Full Names and Explanations
- **DCGM (Data Center GPU Manager)**: NVIDIA's toolkit for monitoring, managing, and configuring GPU systems and data center workloads.
- **Prometheus**: An open-source monitoring and alerting toolkit designed for reliability and scalability, commonly used with time-series metrics.
- **Grafana**: An open-source platform for monitoring and observability, providing dashboards for metrics collected by Prometheus and other sources.

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
