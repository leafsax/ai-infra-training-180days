### Day 123: Network Topologies for AI Training (Fat-Tree, Dragonfly, Torus)

**1) Topic and Core Examination Areas**
- Topic: Network Topologies for AI Training.
- Core Examination Areas: Understanding common network topologies (Fat-Tree, Dragonfly, Torus/3D Torus), their scalability, bisection bandwidth, and fault tolerance.

**2) Requirement Clarification and Metric Definitions**
- **Bisection Bandwidth**: The minimum bandwidth available to cut the network into two halves. For AI training, bisection bandwidth should match or exceed aggregate compute bandwidth (e.g., for 64x 800G GPUs, bisection bandwidth >= 32x 800G).
- **Network Diameter**: The maximum number of hops between any two nodes. Target diameter for Fat-Tree is 3 or 5 hops.
- **Scale**: Number of GPUs supported (e.g., 8K, 32K GPUs).

**3) Core Architecture/Technical Component Design**
- *Fat-Tree Topology*: A multi-tier switch architecture (leaf, spine, core) ensuring non-blocking communication and equal-cost multi-path (ECMP) routing.
- *Dragonfly Topology*: A hierarchical topology with high-degree routers, optimizing for large-scale clusters by reducing the number of switches needed.
- *3D Torus Topology*: Used in supercomputers (e.g., Summit, Frontier), where nodes are arranged in a 3D grid and connected to neighbors, optimizing for locality.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Fat-Tree*: Ensures bisection bandwidth equals total port bandwidth. Requires O(N^2) switches for N leaf nodes, making it expensive at massive scale but optimal for All-Reduce performance.
- *Dragonfly*: Reduces switch count by using global and local groups. Can suffer from global congestion if not managed with adaptive routing and congestion control.
- *Torus*: Excellent locality and low diameter for supercomputing, but less flexible for dynamic cluster scaling compared to Fat-Tree.

**5) Trade-off analysis**
- *Fat-Tree vs Dragonfly*: Fat-Tree offers predictable, non-blocking performance but scales poorly in switch count. Dragonfly scales better in switch count but requires sophisticated routing and congestion control to avoid global congestion.
- *Torus vs Tree*: Torus is great for tightly coupled HPC workloads with regular communication patterns, but AI training (All-Reduce across all nodes) benefits more from Fat-Tree's full bisection bandwidth.

**6) How to determine the optimal solution**
- For standard AI clusters up to a few thousand GPUs, Fat-Tree (e.g., 3-tier or 5-tier) is optimal. For 10K+ GPUs, Dragonfly or folded Clos topologies with adaptive routing are considered to manage switch count and cost.

**7) Full names and explanations of all nouns and abbreviations**
- **Bisection Bandwidth**: The minimum total bandwidth of links that must be cut to divide a network into two equal halves.
- **ECMP**: Equal-Cost Multi-Path — a routing strategy that utilizes multiple available paths of a similar cost to a destination.
- **Clos Network**: A multistage switch network topology, the Fat-Tree is a specific type of Clos network.
- **Hops**: The number of network devices (routers/switches) a packet passes through from source to destination.

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
