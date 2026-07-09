### **Day 80: Request Queuing, Backpressure, and Rate Limiting in AI Systems**

**1) Topic and Core Examination Areas**
- Request queuing strategies for AI serving
- Backpressure mechanisms
- Rate limiting and quota management

**2) Requirement Clarification and Metric Definitions**
- Queue capacity: 10,000 pending requests
- Queue timeout: 30 seconds
- Rate limit: 100 requests/second per tenant
- Backpressure trigger: When queue length > 80% of capacity

**3) Core Architecture/Technical Component Design**
- Queue Layer: Redis queue, Kafka, or in-memory queue (in serving framework)
- Rate Limiter: Token bucket or leaky bucket algorithm, implemented in API gateway or serving layer
- Backpressure: Reject new requests or return 429 Too Many Requests when queue is full

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Token Bucket vs Leaky Bucket**: Token bucket allows bursts up to a limit, then rates at a constant speed. Leaky bucket processes requests at a constant rate, smoothing out bursts.
- **Queue in Gateway vs Queue in Serving**: Gateway queue (e.g., API gateway) provides a unified entry point and can rate-limit across models. Serving-level queue (e.g., vLLM's queue) is optimized for model-specific batching.

**5) Trade-off analysis**
- Gateway queue: Centralized control, but adds latency and a single point of failure.
- Serving queue: Lower latency, integrated with batching, but harder to enforce cross-model rate limits.

**6) How to determine the optimal solution**
- For multi-tenant AI platforms, implement rate limiting at the API gateway level.
- For high-throughput single model serving, rely on the serving framework's internal queue and continuous batching.
- Use backpressure (429 errors) to protect the system from overload.

**7) Full names and explanations of nouns and abbreviations**
- **Backpressure**: A mechanism where a system signals to upstream components to slow down or stop sending data when it is overwhelmed.
- **Rate Limiting**: Restricting the number of requests a user or tenant can make in a given time period.
- **Token Bucket**: A rate limiting algorithm that allows bursts up to a bucket capacity, then limits to a refill rate.
- **Leaky Bucket**: A rate limiting algorithm that processes requests at a constant rate, queuing excess requests.

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
