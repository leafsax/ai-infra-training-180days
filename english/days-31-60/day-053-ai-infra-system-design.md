## Day 53: Energy Efficiency and Cooling in AI Data Centers

### 1) Topic and Core Examination Areas
**Topic**: Power and Cooling Infrastructure for AI Clusters.
**Core Examination Areas**: Power density, liquid cooling, and energy efficiency metrics (PUE).

### 2) Requirement Clarification and Metric Definitions
- **Power per Rack**: e.g., traditional racks: 10-20 kW; AI racks (8x H100): 120-240 kW.
- **PUE (Power Usage Effectiveness)**: Ratio of total facility power to IT equipment power. Target: < 1.2 for modern data centers.
- **Liquid Cooling**: Direct-to-chip or immersion cooling to handle high heat density.

### 3) Core Architecture/Technical Component Design
- **Power Distribution Units (PDUs)**: High-capacity PDUs to deliver power to AI racks.
- **Cooling Systems**: Air cooling (limited to ~30 kW/rack), liquid cooling (direct-to-chip cold plates or immersion).

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Air Cooling**: Standard, but insufficient for 100+ kW racks.
- **Liquid Cooling (Cold Plates)**: Directs coolant through plates attached to GPUs and CPUs. Effective for 120-240 kW/rack.
- **Immersion Cooling**: Submerges servers in dielectric fluid. Highest efficiency, but complex deployment and maintenance.

### 5) Trade-off Analysis
- **Air Cooling**: Lower upfront cost, but cannot support high-density AI racks.
- **Liquid Cooling**: Higher upfront cost and infrastructure change, but necessary for H100/Blackwell clusters and improves PUE.

### 6) How to Determine the Optimal Solution
For clusters with 8x H100 or Blackwell GPUs per rack, liquid cooling (cold plates) is the optimal and often mandatory solution.

### 7) Glossary: Full Names and Explanations
- **PUE (Power Usage Effectiveness)**: A metric measuring the efficiency of a data center, calculated as total facility power divided by IT equipment power. Lower is better (ideal is 1.0).
- **Cold Plate Cooling**: A liquid cooling method where coolant flows through plates attached directly to heat-generating components like GPUs and CPUs.
- **Immersion Cooling**: A cooling technique where servers are submerged in a thermally conductive but electrically insulating dielectric fluid.

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
