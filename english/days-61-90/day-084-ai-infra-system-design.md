### **Day 84: Fault Tolerance and High Availability in AI Serving Systems**

**1) Topic and Core Examination Areas**
- High availability (HA) design for model serving
- Fault tolerance and retry mechanisms
- Multi-region and multi-zone deployment

**2) Requirement Clarification and Metric Definitions**
- Availability target: 99.99%
- RTO (Recovery Time Objective): < 5 minutes
- RPO (Recovery Point Objective): 0 for stateless serving; < 1 hour for model artifacts

**3) Core Architecture/Technical Component Design**
- Multi-Zone Deployment: Spread pods across availability zones
- Health Checks: Liveness and readiness probes for serving pods
- Failover: Automatic routing to healthy replicas or zones

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Stateless vs Stateful Serving**: LLM serving is mostly stateless (except KV Cache in memory). Statelessness enables easy scaling and failover. KV Cache is ephemeral and lost on pod restart, but this is acceptable as it only affects in-flight requests.
- **Liveness vs Readiness Probes**: Liveness probe determines if the pod should be restarted. Readiness probe determines if the pod should receive traffic.

**5) Trade-off analysis**
- Multi-zone: Increases HA but adds cross-zone network latency and cost.
- Stateless design: Simplifies scaling and failover, but in-flight KV Cache is lost on restart.

**6) How to determine the optimal solution**
- Deploy model serving pods across multiple availability zones for HA.
- Use readiness and liveness probes to ensure only healthy pods receive traffic.
- Accept KV Cache loss on restart as in-flight requests will be retried by clients or queue.

**7) Full names and explanations of nouns and abbreviations**
- **HA**: High Availability. A system design that ensures a high level of operational performance.
- **RTO**: Recovery Time Objective. The targeted duration of time within which a business process must be restored after a disaster.
- **RPO**: Recovery Point Objective. The maximum targeted period in which data might be lost from an IT service due to a major incident.
- **Liveness/Readiness Probes**: Kubernetes mechanisms to check if a container is running (liveness) or ready to serve traffic (readiness).

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
