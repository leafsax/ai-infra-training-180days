### **Day 83: Multi-Model Serving and Tenant Isolation**

**1) Topic and Core Examination Areas**
- Multi-model serving architectures
- Tenant isolation (data, compute, model)
- Resource allocation and quotas

**2) Requirement Clarification and Metric Definitions**
- Number of models served: 50+ models on same cluster
- Tenant isolation: Compute isolation via GPU assignment, data isolation via separate vector DBs
- QPS per tenant: 1,000 QPS max

**3) Core Architecture/Technical Component Design**
- Multi-Model Serving: KServe, vLLM with multiple model instances, or model router
- Tenant Routing: L7 load balancer routes by tenant ID to specific model endpoints
- Resource Quotas: Kubernetes namespaces with resource limits per tenant

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Shared vs Isolated Serving**: Shared serving (multi-tenant on same GPUs) is cost-effective but requires careful isolation and QoS guarantees. Isolated serving (dedicated GPUs per tenant) is secure and performant but costly.
- **Model Router**: A layer that routes incoming requests to the appropriate model instance based on tenant, model version, or priority.

**5) Trade-off analysis**
- Shared serving: High utilization, lower cost, but risk of noisy neighbor effects.
- Isolated serving: Predictable performance and security, but lower utilization and higher cost.

**6) How to determine the optimal solution**
- For cost-sensitive, similar-workload tenants, use shared serving with QoS limits and monitoring.
- For enterprise tenants with security or performance SLAs, use isolated serving with dedicated GPU nodes.

**7) Full names and explanations of nouns and abbreviations**
- **QoS**: Quality of Service. A mechanism to guarantee performance levels for different types of traffic or workloads.
- **Noisy Neighbor**: A scenario where one tenant's workload negatively impacts the performance of another tenant on shared resources.

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
