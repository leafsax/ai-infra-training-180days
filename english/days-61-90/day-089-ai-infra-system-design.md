### **Day 89: Edge AI Auto-Scaling and Distributed Inference**

**1) Topic and Core Examination Areas**
- Edge AI inference and deployment
- Distributed inference across edge and cloud
- Edge auto-scaling and device management

**2) Requirement Clarification and Metric Definitions**
- Edge devices: 10,000 IoT devices or edge GPUs
- Inference latency target at edge: < 100 ms
- Cloud offload threshold: When edge GPU utilization > 90%

**3) Core Architecture/Technical Component Design**
- Edge Inference: ONNX Runtime, TensorRT, or llama.cpp on edge GPUs/CPU
- Cloud Offload: Route complex queries or high-load periods to cloud LLM serving
- Device Management: Kubernetes KubeEdge or AWS IoT Greengrass

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Edge vs Cloud Inference**: Edge inference reduces latency and bandwidth cost, but edge devices have limited compute. Cloud inference offers powerful models but adds network latency.
- **Distributed Inference**: Splitting the inference workload or model layers across edge and cloud (e.g., embedding at edge, generation in cloud).

**5) Trade-off analysis**
- Edge inference: Low latency, privacy, offline capability, but limited model size and compute.
- Cloud offload: Powerful models, but network latency and cost.

**6) How to determine the optimal solution**
- For simple, latency-sensitive tasks (e.g., local embedding, filtering), use edge inference.
- For complex generation or RAG, offload to cloud serving when edge capacity is exceeded or task complexity is high.

**7) Full names and explanations of nouns and abbreviations**
- **Edge AI**: AI inference performed on local devices (edge) rather than in centralized cloud data centers.
- **ONNX Runtime**: Open Neural Network Exchange Runtime. A cross-platform inference accelerator.
- **KubeEdge**: An open-source system for extending native containerized application orchestration to edge devices.

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
