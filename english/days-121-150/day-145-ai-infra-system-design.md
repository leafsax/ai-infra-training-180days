### Day 145: Job Placement Algorithms: Topology-aware Scheduling

**1) Topic and Core Examination Areas**
- Topic: Job Placement Algorithms: Topology-aware Scheduling.
- Core Examination Areas: Understanding how to place AI training Pods on nodes to minimize network communication latency and maximize bandwidth, considering GPU, NIC, and switch topologies.

**2) Requirement Clarification and Metric Definitions**
- **Placement Latency Impact**: Poor placement (e.g., GPUs on different switches) can increase All-Reduce latency by 2-5x compared to optimal placement (same switch or NVLink domain).
- **Topology Awareness**: The scheduler should be aware of node-to-switch mappings, NVLink domains, and RDMA network zones.

**3) Core Architecture/Technical Component Design**
- *Node Affinity and Anti-Affinity*: Kubernetes scheduling rules that place Pods on or avoid specific nodes based on labels (e.g., `topology.kubernetes.io/zone`, `rack`).
- *Topology Spread Constraints*: Ensure Pods of a gang job are spread across failure domains (zones, racks) for fault tolerance, or concentrated in the same domain for performance.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *NVLink Domain Awareness*: Schedule all 8 GPUs of an 8-GPU training job on the same node to leverage NVLink full-mesh bandwidth.
- *RDMA Network Zone Awareness*: Schedule inter-node communication partners on nodes that share the same Top-of-Rack (ToR) switch or IB partition to minimize network hops.

**5) Trade-off analysis**
- *Performance vs Fault Tolerance*: Concentrating a job on a single node or switch maximizes performance (NVLink, same ToR) but creates a single point of failure. Spreading Jobs across racks or zones improves fault tolerance but may increase network latency.
- *Complexity of Topology Awareness*: Topology-aware schedulers require detailed cluster topology information and can be more complex to configure and maintain than default schedulers.

**6) How to determine the optimal solution**
- For performance-critical LLM training, use topology-aware scheduling to place all GPUs of a job on the same node (for intra-node) or within the same ToR switch domain (for inter-node). Use topology spread constraints for fault-tolerant placement across zones when job length justifies the latency trade-off.

**7) Full names and explanations of all nouns and abbreviations**
- **Node Affinity**: A Kubernetes scheduling feature that constrains which nodes a Pod is eligible to be scheduled on, based on labels.
- **ToR**: Top-of-Rack — the network switch at the top of a server rack, connecting all servers within that rack.
- **IB Partition**: InfiniBand partition key (P_Key) used to isolate and manage communication domains within an IB fabric.

---

