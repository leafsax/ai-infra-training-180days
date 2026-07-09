### **Day 78: GPU Auto-Scaling and Node Provisioning (Cluster Autoscaler, Karpenter)**

**1) Topic and Core Examination Areas**
- GPU node provisioning and auto-scaling
- Cluster Autoscaler vs Karpenter
- Spot instances and GPU cost optimization

**2) Requirement Clarification and Metric Definitions**
- GPU node types: NVIDIA A10G, A100, H100
- Provisioning latency target: < 3 minutes for new GPU node
- Spot instance usage target: 70% of non-critical workloads

**3) Core Architecture/Technical Component Design**
- Node Auto-Scaler: Cluster Autoscaler or Karpenter
- Instance Types: Managed GPU instances (AWS EC2 P3/P4/G5, GCP A2, Azure NDm)
- Spot Integration: Spot fleet or Karpenter spot strategies

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Cluster Autoscaler vs Karpenter**: Cluster Autoscaler is the native Kubernetes auto-scaler, but is slower and less flexible. Karpenter is a newer, node-level auto-scaler that provisions nodes in seconds based on pod requirements, supporting spot instances and diverse instance types.
- **On-Demand vs Spot GPU instances**: Spot instances are 60-80% cheaper but can be reclaimed with short notice. On-demand is stable but expensive.

**5) Trade-off analysis**
- Cluster Autoscaler: Mature and stable, but slow provisioning (minutes) and less instance type flexibility.
- Karpenter: Fast provisioning (seconds), intelligent instance selection, but requires setup and is less mature than CA.
- Spot vs On-Demand: Spot saves cost but introduces interruption risk; on-demand is reliable but costly.

**6) How to determine the optimal solution**
- For fast, intelligent GPU node provisioning, choose Karpenter.
- For non-critical, fault-tolerant AI workloads (e.g., batch embedding generation), use Spot instances.
- For production model serving, use On-Demand or Spot with graceful degradation and retry logic.

**7) Full names and explanations of nouns and abbreviations**
- **Cluster Autoscaler**: A Kubernetes component that adds or removes nodes from the cluster based on unschedulable pods.
- **Karpenter**: An open-source node provisioner project for Kubernetes that provides fast, flexible node provisioning.
- **Spot Instances**: Unused compute capacity available at a discounted price, which can be reclaimed by the cloud provider.

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


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On RAG Retrieval & Consistency:** In your RAG architecture, how do you handle embedding drift or vector database consistency during document updates? What is the exact latency budget allocated for the retrieval step vs. the LLM generation step to meet a TTFT < 200ms SLA?
- **On Data Pipeline Throughput:** You mentioned ETL/ELT pipelines with Spark/Flink. How do you prevent the data loader from becoming the bottleneck during training? What is the strategy for handling skewed data distributions that cause certain GPU workers to starve?
- **On Auto-Scaling Edge Cases:** For custom metrics-based scaling (e.g., KV cache utilization), what happens if the metrics server itself becomes unavailable? How do you design a fallback mechanism to prevent infinite scaling or sudden pod termination?
