## Day 18: Orchestration and Scheduling (Kubernetes, Ray, Slurm)

### 1) Topic and Core Examination Areas
- **Topic**: Orchestration and Scheduling for AI Workloads.
- **Core Examination Areas**: Kubernetes for serving, Slurm for HPC training, Ray for distributed AI, and GPU scheduling.

### 2) Requirement Clarification and Metric Definitions
- **Scheduling Latency**: Time to provision a GPU node and deploy a model. Target < 5 minutes for serving replicas.
- **Resource Request Accuracy**: Correctly requesting GPU memory, CPU, and network bandwidth to avoid over-provisioning or starvation.

### 3) Core Architecture/Technical Component Design
- **Kubernetes (K8s)**: Container orchestration platform, widely used for LLM serving deployments (via operators like KubeFlow, Ray K8s operator).
- **Slurm**: Workload manager for HPC and AI training clusters, optimizing for long-running batch jobs.
- **Ray**: A distributed framework that includes Ray Serve for model serving and Ray Train for distributed training.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **K8s vs. Slurm for Training**: K8s is more dynamic and suited for serving and short-lived jobs. Slurm is optimized for HPC batch scheduling and long training jobs with complex node topologies.
- **GPU Scheduling Plugins**: K8s plugins (e.g., NVIDIA GPU Operator, Device Plugin) to expose and manage GPUs as resources.

### 5) Trade-off analysis
- **Flexibility vs. Optimization**: Kubernetes offers great flexibility, auto-scaling, and ecosystem support for serving but requires more operational overhead for large-scale training. Slurm is highly optimized for HPC training workloads but less suited for dynamic serving workloads.

### 6) How to determine the optimal solution
- Use Kubernetes (with GPU operators and serving-specific operators like vLLM K8s operators) for LLM inference serving and microservices. Use Slurm or Ray for large-scale distributed training workloads.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Kubernetes (K8s)**: An open-source container orchestration platform for automating deployment, scaling, and management of containerized applications.
- **Slurm**: A popular open-source workload manager and job scheduler for HPC and AI clusters.
- **Ray**: A distributed execution framework for Python and AI workloads, including Ray Train (training) and Ray Serve (serving).
- **GPU Operator**: A Kubernetes operator by NVIDIA that simplifies the management of NVIDIA GPU resources in K8s clusters.

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
