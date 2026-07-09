### Day 140: Unified Storage Architectures for Training and Inference

**1) Topic and Core Examination Areas**
- Topic: Unified Storage Architectures for Training and Inference.
- Core Examination Areas: Designing storage systems that serve both AI training (high throughput, large files) and inference (low latency, high concurrency) workloads efficiently.

**2) Requirement Clarification and Metric Definitions**
- **Training Throughput**: 10-100 GB/s for data loading and checkpoints.
- **Inference Latency (TTFT/TP99)**: For serving, TTFT (Time to First Token) should be <100 milliseconds for interactive chat, and TP99 latency per request <1 second.
- **Inference Concurrency**: Object storage or file systems serving inference model weights should support 1000+ concurrent model loads without metadata bottlenecks.

**3) Core Architecture/Technical Component Design**
- *Unified Data Lake*: A single storage backend (e.g., S3-compatible object storage or distributed file system) that stores raw data, training datasets, model artifacts, and inference serving models.
- *Storage Caching for Inference*: Inference nodes cache model weights in local NVMe or host RAM to reduce repeated storage reads during request serving.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Training Storage vs Inference Storage*: Training storage emphasizes high sequential throughput for data loading and checkpointing. Inference storage emphasizes low-latency metadata operations and concurrent model weight delivery to serving nodes.
- *Model Registry and Storage Integration*: Tools like MLflow or Weights & Biases integrate with storage systems to version model artifacts and ensure inference servers load the correct checkpoint versions.

**5) Trade-off analysis**
- *Shared vs Separate Storage*: A unified storage system simplifies management and ensures data consistency between training and inference but must be tuned to handle both high-throughput (training) and low-latency (inference) workloads. Separate storage systems can be optimized for each workload but increase operational complexity.
- *Object Storage vs File System for Models*: Object storage is ideal for model artifact versioning and archival. For active inference serving, a fast file system or cached object storage gateway is preferred to ensure low-latency model loading.

**6) How to determine the optimal solution**
- Use a unified S3-compatible object storage or distributed file system as the single source of truth for all AI assets (data, checkpoints, models). Implement caching layers (NVMe or RAM) on inference nodes to ensure low-latency model serving. Use model registry tools to version and track artifacts.

**7) Full names and explanations of all nouns and abbreviations**
- **TTFT**: Time to First Token — the latency from sending a prompt to receiving the first generated token in LLM inference.
- **MLflow**: An open-source platform for managing the ML lifecycle, including experimentation, reproducibility, and deployment.
- **Weights & Biases (W&B)**: A platform for tracking and visualizing machine learning experiments and model artifacts.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Task Scheduling)**:
  ```mermaid
  graph TD
      A[User Job Submission] --> B[Scheduler K8s/Slurm]
      B --> C[Gang Scheduling Controller]
      C --> D[Resource Affinity Checker]
      D --> E[GPU Node Allocation]
      E --> F[Job Execution]
  ```

- **Data Flow Diagram (Scheduling Flow)**:
  ```mermaid
  flowchart LR
      A[Job Request] --> B[Queue & Priority Sort]
      B --> C[Resource Availability Check]
      C --> D[Allocate Gang of Pods]
      D --> E[Start Training Job]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Network Reliability & PFC Storms:** For RoCEv2 with DCQCN or InfiniBand, how do you prevent PFC (Priority Flow Control) storms when congestion occurs? What is the exact recovery time and data loss scenario when an 800G NIC or switch link flaps unexpectedly?
- **On Distributed Storage I/O:** For distributed file systems like Lustre/GPFS/Ceph, how do you handle small random I/O patterns from multiple GPU nodes simultaneously? What is the metadata server bottleneck, and how do you scale it without introducing single points of failure?
- **On Gang Scheduling & Preemption:** For Gang Scheduling in K8s/Slurm, what happens if one node in the required gang is unhealthy or evicted? How do you handle preemption fairness across different tenant queues to prevent starvation of long-running training jobs?
