## Day 57: AI Model Registry and MLOps Pipeline Infrastructure

### 1) Topic and Core Examination Areas
**Topic**: MLOps and Model Registry Infrastructure.
**Core Examination Areas**: Model versioning, artifact storage, and CI/CD for ML models.

### 2) Requirement Clarification and Metric Definitions
- **Model Artifacts**: Checkpoints, quantized versions, configuration files.
- **Model Registry**: Centralized repository for model versions, metadata, and deployment status.
- **CI/CD for ML**: Automated testing, validation, and deployment of model versions.

### 3) Core Architecture/Technical Component Design
- **Model Storage**: Object storage (S3) or distributed filesystem for model weights.
- **Model Registry Service**: e.g., MLflow Model Registry, Hugging Face Hub, or custom registry.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **MLflow Model Registry**: Open-source platform for managing the ML lifecycle, including model versioning and staging.
- **Hugging Face Hub**: Community and enterprise platform for hosting, versioning, and sharing models and datasets.

### 5) Trade-off Analysis
- **Custom Registry**: Tailored to specific needs, but requires development and maintenance.
- **Managed Services (MLflow/HF Hub)**: Faster to deploy, rich features, but may have limitations or costs.

### 6) How to Determine the Optimal Solution
For most organizations, MLflow Model Registry or Hugging Face Hub (for open-source or private HF Enterprise) provides the optimal balance of features and ease of use.

### 7) Glossary: Full Names and Explanations
- **MLOps (Machine Learning Operations)**: The practice of applying DevOps principles to machine learning workflows, including training, deployment, and monitoring.
- **Model Registry**: A centralized system for storing, versioning, and managing machine learning models and their metadata.
- **MLflow**: An open-source platform for managing the end-to-end machine learning lifecycle, including tracking, packaging, and deployment.

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
