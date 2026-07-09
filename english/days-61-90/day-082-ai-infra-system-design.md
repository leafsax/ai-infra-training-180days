### **Day 82: Cost Optimization in AI Infra Auto-Scaling (Spot instances, right-sizing)**

**1) Topic and Core Examination Areas**
- GPU cost optimization strategies
- Right-sizing model serving workloads
- Spot instance strategies for AI

**2) Requirement Clarification and Metric Definitions**
- GPU cost target: < $2 per 1M tokens generated
- Right-sizing accuracy: GPU utilization 60-80%
- Spot interruption rate target: < 5% for serving workloads (use on-demand for serving)

**3) Core Architecture/Technical Component Design**
- Monitoring: Cost tracking via cloud provider tools or Kubecost
- Right-sizing: Analyze GPU memory and compute utilization to select appropriate instance types (e.g., A10G vs A100)
- Spot Integration: Use spot for batch jobs, embedding generation, or non-critical training

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Instance Right-Sizing**: Over-provisioned GPUs waste money; under-provisioned GPUs cause latency. Use profiling tools to determine optimal GPU type and memory for the model.
- **Spot vs On-Demand for Serving**: Serving workloads require stability; spot interruptions can cause request failures. Use on-demand for serving, spot for offline batch tasks.

**5) Trade-off analysis**
- Spot instances: 60-80% cost savings, but risk of interruption.
- Right-sizing vs Over-provisioning: Right-sizing optimizes cost but requires continuous monitoring; over-provisioning ensures performance but wastes cost.

**6) How to determine the optimal solution**
- Use on-demand GPU instances for production model serving to ensure stability.
- Use spot instances for batch embedding, data processing, or training workloads that can tolerate interruptions.
- Continuously profile and right-size model serving workloads to maintain 60-80% GPU utilization.

**7) Full names and explanations of nouns and abbreviations**
- **Kubecost**: A tool for monitoring and optimizing Kubernetes cluster costs.
- **Right-sizing**: The practice of matching resource provision (CPU, GPU, memory) to actual workload requirements to optimize cost and performance.

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
