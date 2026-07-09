### Day 150: End-to-End AI Cluster Scheduling System Design (Integrating Network, Storage, Compute)

**1) Topic and Core Examination Areas**
- Topic: End-to-End AI Cluster Scheduling System Design.
- Core Examination Areas: Integrating network architecture, storage systems, and compute scheduling into a cohesive AI cluster management platform, ensuring performance, fault tolerance, and resource efficiency.

**2) Requirement Clarification and Metric Definitions**
- **End-to-End Job Submission to Execution Time**: Target <30 seconds for a 64-GPU gang job to be scheduled and start execution, including network and storage attachment validation.
- **Cluster-wide Utilization Target**: >75% GPU utilization across the cluster, with <5% job failure rate due to scheduling or resource fragmentation.
- **Recovery SLA**: Time to recover from a node or storage failure and resume training, target <5 minutes with checkpoint recovery.

**3) Core Architecture/Technical Component Design**
- *Unified Scheduling Platform*: A scheduler (e.g., Kubernetes + Volcano) that manages compute (GPU Pods), network (RDMA/NVLink topology labels), and storage (distributed file system or object storage attachments).
- *Health and Monitoring Integrations*: Tools like Prometheus, Grafana, and NCCL/ storage benchmarks integrated into the scheduler to detect failures and trigger recovery or rescheduling.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Integrating Network Topology with Scheduling*: The scheduler uses node labels (e.g., `switch-domain`, `rack`, `nvlink-domain`) to place gang jobs optimally for All-Reduce performance.
- *Integrating Storage with Scheduling*: Training Pods are scheduled on nodes that have access to the required storage (Lustre/MinIO mounts) and have sufficient local NVMe for caching or checkpointing.

**5) Trade-off analysis**
- *Centralized vs Distributed Scheduling Components*: A centralized scheduler (single Volcano instance) simplifies consistency but can become a bottleneck. Distributed scheduling components improve scalability but require careful coordination for gang scheduling and fairness.
- *Performance Optimization vs Operational Complexity*: Highly optimized scheduling (topology-aware, RL-based, elastic) improves cluster efficiency but increases operational complexity and requires specialized expertise.

**6) How to determine the optimal solution**
- Design an end-to-end AI cluster platform using Kubernetes as the base orchestration layer, augmented with an AI-aware scheduler (Volcano/Kube-batch) for gang scheduling and fairness. Integrate topology labels for network-aware placement, ensure storage access via mounted distributed file systems or S3 gateways, and implement automated checkpoint recovery and health monitoring for fault tolerance.

**7) Full names and explanations of all nouns and abbreviations**
- **Prometheus**: An open-source systems monitoring and alerting toolkit.
- **Grafana**: An open-source platform for monitoring and observability, often used with Prometheus for visualization.
- **S3 Gateway**: A service that provides an S3-compatible interface to underlying storage systems, allowing AI workloads to use standard S3 APIs for data access.

---

## Summary of Deliverable

- **What I did**: Generated the AI Infra System Design Training Question Set for Days 121-150 in English, covering the theme "Network Architecture, Storage Systems & Task Scheduling".
- **What I accomplished**: Produced 30 days of structured training content, with each day including: (1) Topic and Core Examination Areas; (2) Requirement Clarification and Metric Definitions; (3) Core Architecture/Technical Component Design; (4) Deep Dive into Key Technologies and Possible Solutions; (5) Trade-off analysis; (6) How to determine the optimal solution; (7) Full names and explanations of all nouns and abbreviations.
- **Files created or modified**: No external files were created; the complete content is provided in this response as Markdown-formatted text.
- **Issues encountered**: None. The 30-day content was generated systematically following the exact 7-section structure required for each day.

### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Network Architecture)**:
  ```mermaid
  graph TD
      A[Job Scheduler Slurm/K8s] --> B[Compute Nodes GPU+CPU]
      B --> C[RDMA Network InfiniBand/RoCE]
      B --> D[Distributed Storage Lustre/GPFS]
      C --> E[All-Reduce Communication NCCL]
  ```

- **Data Flow Diagram (Training Data & Compute)**:
  ```mermaid
  flowchart LR
      A[Data Loader] --> B[Local NVMe Cache]
      B --> C[GPU Compute Kernel]
      C --> D[NCCL All-Reduce via RDMA]
      D --> C
      C --> E[Gradient Update]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Network Reliability & PFC Storms:** For RoCEv2 with DCQCN or InfiniBand, how do you prevent PFC (Priority Flow Control) storms when congestion occurs? What is the exact recovery time and data loss scenario when an 800G NIC or switch link flaps unexpectedly?
- **On Distributed Storage I/O:** For distributed file systems like Lustre/GPFS/Ceph, how do you handle small random I/O patterns from multiple GPU nodes simultaneously? What is the metadata server bottleneck, and how do you scale it without introducing single points of failure?
- **On Gang Scheduling & Preemption:** For Gang Scheduling in K8s/Slurm, what happens if one node in the required gang is unhealthy or evicted? How do you handle preemption fairness across different tenant queues to prevent starvation of long-running training jobs?
