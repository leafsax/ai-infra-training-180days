### **Day 77: Kubernetes Auto-Scaling (HPA, VPA, KEDA, CRDs)**

**1) Topic and Core Examination Areas**
- Kubernetes scaling mechanisms: HPA, VPA, KEDA
- Custom Resource Definitions (CRDs) for AI workloads
- Metric-based auto-scaling

**2) Requirement Clarification and Metric Definitions**
- Pod scaling range: 2 to 20 replicas
- VPA recommendations: CPU 4 cores, Memory 16 GB, GPU 1 per pod
- Scaling latency target: < 30 seconds from trigger to new pod ready

**3) Core Architecture/Technical Component Design**
- HPA: Scales based on CPU, memory, or custom metrics (via Custom Metrics API)
- VPA: Vertical Pod Autoscaler, recommends or applies resource requests/limits
- KEDA: Kubernetes Event-driven Autoscaling, scales based on external events (Kafka queue length, Prometheus metrics)

**4) Deep Dive into Key Technologies and Possible Solutions**
- **HPA vs KEDA**: HPA is built into Kubernetes and scales based on resource metrics or custom metrics. KEDA is an operator that scales based on event sources (message queues, HTTP concurrency) and is more suitable for bursty AI workloads.
- **VPA vs Manual Resource Tuning**: VPA automatically recommends or sets resource requests, but can cause pod evictions and restarts. Manual tuning requires expertise but offers stability.

**5) Trade-off analysis**
- HPA: Standard, widely supported, but may not natively support GPU or custom AI metrics without setup.
- KEDA: Excellent for event-driven scaling (e.g., queue length), but requires additional operator and configuration.
- VPA: Optimizes resource usage but can cause instability due to pod rescheduling.

**6) How to determine the optimal solution**
- For standard resource-based scaling (CPU/memory), use HPA.
- For event-driven or custom metric scaling (GPU utilization, queue length), use KEDA.
- For resource optimization over time, use VPA in recommendation mode, not auto-apply, to avoid disruptions.

**7) Full names and explanations of nouns and abbreviations**
- **VPA**: Vertical Pod Autoscaler. A Kubernetes tool that adjusts the CPU and memory resources of pods.
- **KEDA**: Kubernetes Event-driven Autoscaling. An open-source project for event-driven scale.
- **CRD**: Custom Resource Definition. A way to extend the Kubernetes API with custom resources.
- **Custom Metrics API**: Kubernetes API for exposing custom metrics for HPA.

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
