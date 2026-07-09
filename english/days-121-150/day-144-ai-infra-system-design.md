### Day 144: Multi-tenant Scheduling and Resource Isolation

**1) Topic and Core Examination Areas**
- Topic: Multi-tenant Scheduling and Resource Isolation.
- Core Examination Areas: Understanding how to schedule AI workloads for multiple teams or users (tenants) while ensuring fair resource allocation and isolation to prevent noisy neighbor effects.

**2) Requirement Clarification and Metric Definitions**
- **Tenant Isolation**: Ensure that one tenant's workload does not degrade another tenant's performance beyond a defined SLA (e.g., <10% latency degradation).
- **Fair Sharing Metrics**: Share metrics like "share target" (e.g., each tenant gets 20% of cluster GPUs) and "boost factor" (priority weighting for certain tenants).
- **Quota Limits**: Maximum number of GPUs or CPU cores a tenant can request simultaneously.

**3) Core Architecture/Technical Component Design**
- *Namespaces and Quotas*: Kubernetes namespaces with ResourceQuota and LimitRange objects to enforce per-tenant resource limits.
- *Fair Share Schedulers*: Schedulers like Kube-Fairness or Yunikorn implement fair sharing algorithms (e.g., Dominant Resource Fairness - DRF) to allocate GPUs based on tenant claims and priorities.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Dominant Resource Fairness (DRF)*: A fair allocation algorithm that considers the dominant resource (e.g., GPUs vs CPU) for each job and ensures proportional sharing based on reported demands.
- *Quality of Service (QoS) Classes*: Kubernetes QoS classes (Guaranteed, Burstable, BestEffort) can be used to prioritize eviction and resource allocation during contention.

**5) Trade-off analysis**
- *Strict Quotas vs Flexible Sharing*: Strict quotas prevent overuse but can lead to underutilization if a tenant's quota is not fully used. Flexible sharing (fair share) improves utilization but requires careful tuning to prevent large jobs from starving small ones.
- *Isolation vs Utilization*: Strong resource isolation (MIG, dedicated nodes) ensures performance but reduces overall cluster utilization. Soft isolation (time-slicing, fair share) improves utilization but risks noisy neighbor effects.

**6) How to determine the optimal solution**
- Use Kubernetes namespaces with resource quotas for basic tenant isolation. For advanced fair sharing across multiple teams, deploy a fair-share scheduler like Yunikorn or Kube-Fairness with DRF. Use QoS classes and preemption policies to handle contention fairly.

**7) Full names and explanations of all nouns and abbreviations**
- **DRF**: Dominant Resource Fairness — a fair allocation algorithm that generalizes proportional share to multiple resource types.
- **QoS**: Quality of Service — a Kubernetes classification (Guaranteed, Burstable, BestEffort) that determines how the scheduler and evictor treat Pods during resource contention.
- **Namespace**: A Kubernetes mechanism for dividing cluster resources between multiple users or teams.

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
