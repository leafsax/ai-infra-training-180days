### Day 133: Object Storage for AI Workloads (S3, OCS, MinIO)

**1) Topic and Core Examination Areas**
- Topic: Object Storage for AI Workloads.
- Core Examination Areas: Understanding S3-compatible object storage, its use cases in AI (data lakes, checkpoint archival), and performance considerations.

**2) Requirement Clarification and Metric Definitions**
- **Throughput**: Object storage should support 10+ GB/s aggregate throughput for large file uploads/downloads (e.g., model checkpoints, dataset ingestion).
- **Latency**: Object storage API latency (put/get) for large objects should be <100 milliseconds for initiation, with throughput dominating total time.
- **Durability**: Target 99.999999999% (11 nines) durability for model checkpoints and training data.

**3) Core Architecture/Technical Component Design**
- *S3-Compatible Storage*: Amazon S3, IBM Cloud Object Storage (OCS), or self-hosted MinIO clusters that implement the S3 API.
- *Data Lake Architecture*: Object storage serves as the central repository for raw data, processed datasets, and model artifacts, accessible by training, inference, and MLops pipelines.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *MinIO vs Cloud S3*: MinIO is a high-performance, self-hosted S3-compatible object storage server optimized for AI workloads and on-premises clusters. Cloud S3 offers managed durability and scalability but incurs egress costs and network latency.
- *S3 API vs POSIX*: S3 uses a flat namespace and object-based access (put/get/delete), unlike POSIX file systems with hierarchical directories. AI data loaders must be adapted to use S3 SDKs or FUSE-based S3 mounts (e.g., s3fs, which has performance limitations).

**5) Trade-off analysis**
- *Object Storage vs File Systems for Training Data*: Object storage is excellent for archival and data lake workflows but is not ideal for high-frequency, small-file training data loading due to S3 API latency and lack of POSIX semantics.
- *Self-hosted vs Managed*: Self-hosted MinIO offers lower latency and no egress fees but requires operational overhead. Managed S3 offers durability and scale but at higher cost and potential network bottlenecks.

**6) How to determine the optimal solution**
- Use a high-performance object storage system like MinIO for on-premises data lakes and checkpoint archival. Use cloud S3 for multi-region data storage and long-term archival. For active training data loading, use a distributed file system or a high-performance S3 gateway with caching.

**7) Full names and explanations of all nouns and abbreviations**
- **S3**: Simple Storage Service — Amazon's object storage service, which has become an industry standard API for object storage.
- **OCS**: Object Storage Service — cloud-based object storage offerings (e.g., IBM Cloud Object Storage, Oracle Cloud Storage).
- **FUSE**: Filesystem in Userspace — a software interface for operating systems that allows users to create their own file system without modifying the kernel code.

---



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
