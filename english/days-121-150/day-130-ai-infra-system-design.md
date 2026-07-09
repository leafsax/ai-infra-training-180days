### Day 130: Network Fault Tolerance and Recovery in AI Clusters

**1) Topic and Core Examination Areas**
- Topic: Network Fault Tolerance and Recovery in AI Clusters.
- Core Examination Areas: Understanding network failure modes (NIC failure, switch failure, cable issues), detection mechanisms, and recovery strategies in AI training clusters.

**2) Requirement Clarification and Metric Definitions**
- **MTBF (Mean Time Between Failures)**: Target network component MTBF should be >500,000 hours for switches and NICs.
- **Recovery Time**: Time to detect and recover from a network link failure should be <1 second to minimize training job disruption.
- **Job Completion Rate**: Target >95% job completion rate for long-running training jobs (weeks to months) despite network faults.

**3) Core Architecture/Technical Component Design**
- *Redundant Paths*: Fat-Tree and Clos topologies provide multiple paths between any two nodes. If a link or switch fails, ECMP or adaptive routing can reroute traffic.
- *Health Checking*: RDMA/IB networks use link layer health checks (e.g., IB Link Down/Up events). RoCEv2 networks use PFC/ECN status and ping-like RDMA probes.
- *NCCL Fault Tolerance*: NCCL can detect failed ranks and attempt to reinitialize communication groups, though severe failures may require job restart.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Link Failure Detection*: InfiniBand uses Link Layer Discovery Protocol (LLDP) and hardware link status. RoCEv2 relies on Ethernet link status and RDMA connection management (RC/QP errors).
- *Recovery Strategies*: For transient errors (e.g., temporary congestion causing packet loss), RDMA auto-retransmission handles recovery. For persistent failures (e.g., dead NIC), the scheduler must detect the failed node and restart the job or migrate it to healthy resources.

**5) Trade-off analysis**
- *Automatic Recovery vs Job Restart*: RDMA auto-retransmission handles minor packet loss but cannot recover from link down events. For link/switch failures, job checkpoint recovery or full restart is often required, trading off compute time for fault tolerance.
- *Redundancy Cost vs Reliability*: Extra switch ports and redundant cabling increase cost but significantly improve MTBF and recovery times.

**6) How to determine the optimal solution**
- Design the network topology with sufficient redundancy (e.g., 3-tier Fat-Tree with multiple spines). Implement automated health checking and integrate with the job scheduler for rapid failure detection and checkpoint-based recovery.

**7) Full names and explanations of all nouns and abbreviations**
- **MTBF**: Mean Time Between Failures — a measure of how reliable a hardware or software component is.
- **LLDP**: Link Layer Discovery Protocol — a network protocol used by network devices to advertise their identity, capabilities, and neighbors.
- **QP**: Queue Pair — the fundamental communication structure in RDMA, consisting of a Send Queue and a Receive Queue.
- **ECMP**: Equal-Cost Multi-Path routing — used for fault tolerance and load balancing in case of link failures.

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
