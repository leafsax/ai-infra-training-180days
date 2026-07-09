## Day 41: AI Cluster Networking - Fat-Tree, Dragonfly, and Network Topologies

### 1) Topic and Core Examination Areas
**Topic**: Network Topologies for AI Clusters.
**Core Examination Areas**: Fat-Tree, Clos networks, Dragonfly topology, and bisection bandwidth.

### 2) Requirement Clarification and Metric Definitions
- **Bisection Bandwidth**: The minimum bandwidth required to split the network into two equal halves. Critical for All-Reduce performance.
- **Radix**: The number of ports on a switch.
- **k-ary n-tree**: A fat-tree topology with k ports per switch and n levels.

### 3) Core Architecture/Technical Component Design
- **Fat-Tree Topology**: A multi-level switch fabric where links towards the core have higher aggregate bandwidth than links at the edge, ensuring no congestion during All-Reduce.
- **Spine-Leaf Architecture**: Common Ethernet implementation of fat-tree, where leaf switches connect to servers and spine switches connect leaf switches.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Fat-Tree/Clos Networks**: Provide non-blocking communication, essential for distributed training All-Reduce operations. 
- **Dragonfly Topology**: Used for very large clusters (thousands of nodes), optimizing for fewer switches and longer links, but requires advanced routing algorithms to avoid congestion.

### 5) Trade-off Analysis
- **Fat-Tree**: Excellent performance, non-blocking, but requires many switches and high cabling complexity/cost.
- **Dragonfly**: Scales to tens of thousands of GPUs with fewer switches, but complex routing and potential for local congestion if not tuned properly.

### 6) How to Determine the Optimal Solution
For clusters up to a few thousand GPUs, Fat-Tree (Spine-Leaf) with InfiniBand or RoCEv2 is standard. For extreme scale (10k+ GPUs), Dragonfly or hierarchical topologies are considered.

### 7) Glossary: Full Names and Explanations
- **Fat-Tree**: A network topology where bandwidth increases as you move toward the core switches, ensuring non-blocking communication for all nodes.
- **Bisection Bandwidth**: The total bandwidth available to connect two equal halves of a network; a key metric for evaluating the performance of distributed training workloads.
- **Spine-Leaf Architecture**: A network topology common in data centers where "leaf" switches connect to servers (or compute nodes) and "spine" switches interconnect the leaf switches.
- **Dragonfly Topology**: A network topology designed for very large-scale clusters, using high-radix switches and global groups to minimize the number of switches and links required.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram**:
  ```mermaid
  graph TD
      A[Load Balancer] --> B[Inference Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU Resource Manager]
      D --> E[Kubernetes / Karpenter]
      B --> F[Telemetry DCGM/OpenTelemetry]
  ```

- **Data Flow Diagram**:
  ```mermaid
  flowchart LR
      A[Incoming Requests] --> B[Continuous Batching Scheduler]
      B --> C[KV Cache Lookup & Update]
      C --> D[GPU Tensor Core Compute]
      D --> E[Output Tokens]
      E --> F[Streaming Response]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On KV Cache & PagedAttention:** You proposed PagedAttention for KV cache management. What happens when HBM is heavily fragmented under varying context lengths? How do you handle cache eviction policies under heavy load without degrading model accuracy or causing hallucinations?
- **On GPU Partitioning (MIG/vGPU):** For MIG or vGPU partitioning, what is the hard isolation guarantee? Can a noisy neighbor in one MIG partition still affect another via shared PCIe lanes or CPU memory bandwidth? How do you measure and enforce SLOs per partition in real-time?
- **On Auto-Scaling & Thrashing:** For KEDA-based auto-scaling, what is the scale-from-zero latency for bringing up a new vLLM pod? How do you prevent scaling thrashing when QPS fluctuates rapidly around the scaling threshold (e.g., +20% then -20% within 10 seconds)?
