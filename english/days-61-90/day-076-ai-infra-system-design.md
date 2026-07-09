### **Day 76: Auto-Scaling Fundamentals for AI Serving Systems**

**1) Topic and Core Examination Areas**
- Auto-scaling concepts and triggers (CPU, GPU, memory, custom metrics)
- Scaling up vs scaling out
- Scaling AI model serving workloads

**2) Requirement Clarification and Metric Definitions**
- Current inference QPS: 2,000 QPS, peaking at 10,000 QPS
- GPU utilization target: 60-80% for cost-effective scaling
- Scaling trigger: Scale up when GPU utilization > 75% for 5 minutes
- Scale-in cooldown: 10 minutes to prevent flapping

**3) Core Architecture/Technical Component Design**
- Metrics Collection: Prometheus, GPU metrics (nvml, DCGM)
- Auto-Scaler: Kubernetes HPA (Horizontal Pod Autoscaler), custom scalers for GPU metrics
- Model Serving: vLLM, TensorRT-LLM, or KServe

**4) Deep Dive into Key Technologies and Possible Solutions**
- **CPU-based HPA vs GPU-based HPA**: CPU metrics are standard but may not reflect AI workload pressure (a model can be GPU-bound while CPU is idle). GPU-based scaling (using DCGM or custom metrics) is more accurate for LLM serving.
- **Scaling Pods vs Scaling Nodes**: Scaling pods (horizontal scaling of model instances) is faster but limited by available nodes. Scaling nodes (adding GPU nodes) takes longer (provisioning time) but increases capacity.

**5) Trade-off analysis**
- Pod scaling: Fast, but may hit node capacity limits or cause resource fragmentation.
- Node scaling: Increases capacity but has provisioning latency (minutes for GPU nodes), risking request spikes during scale-up.

**6) How to determine the optimal solution**
- Use pod-level auto-scaling (HPA) for rapid response to QPS changes within existing node capacity.
- Use node-level auto-scaling (Cluster Autoscaler / Karpenter) to handle sustained high load or new model deployments.
- Combine both: HPA for pod scaling, triggered by GPU utilization or queue length.

**7) Full names and explanations of nouns and abbreviations**
- **HPA**: Horizontal Pod Autoscaler. A Kubernetes component that automatically scales the number of pod replicas.
- **GPU**: Graphics Processing Unit. Originally for graphics, now widely used for AI/ML compute.
- **DCGM**: Data Center GPU Manager. NVIDIA's tool for monitoring and managing GPU metrics.
- **nvml**: NVIDIA Management Library. API for monitoring and managing NVIDIA GPUs.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Auto-Scaling)**:
  ```mermaid
  graph TD
      A[API Gateway] --> B[Inference Service vLLM]
      B --> C[Custom Metrics Server]
      C --> D[KEDA Scaler]
      D --> E[Kubernetes Cluster]
      E --> B
  ```

- **Data Flow Diagram (Auto-Scaling Flow)**:
  ```mermaid
  flowchart LR
      A[Request Load] --> B[Metrics Exporter]
      B --> C[Prometheus / DCGM]
      C --> D[KEDA Operator]
      D --> E[Scale In/Out Pods]
  ```
