### Day 157: API Security and Rate Limiting for LLM Services

**1) Topic and Core Examination Areas**
- Topic: API Security and Rate Limiting for Large Language Model (LLM) Services.
- Core Examination Areas: Preventing abuse, DDoS protection, token-based rate limiting, and anomaly detection.

**2) Requirement Clarification and Metric Definitions**
- **QPS**: 20,000 QPS peak capacity.
- **Rate Limit**: 100 requests per minute per user tier; 1,000 requests per minute for enterprise tier.
- **Latency Metrics**: TTFT < 200ms; TP99 < 2 seconds.
- **Abuse Detection**: Identify and block > 95% of malicious scraping or prompt injection attempts within 100ms of request arrival.

**3) Core Architecture/Technical Component Design**
- **API Gateway**: Implements rate limiting, authentication, and initial request validation.
- **Token Bucket Algorithm**: Used for rate limiting to allow burst traffic within defined limits.
- **WAF (Web Application Firewall)**: Filters malicious payloads and known attack patterns.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Solution A: Token Bucket Rate Limiting*: Allows bursts up to a bucket capacity, then regulates at a fixed rate.
- *Solution B: Leaky Bucket Rate Limiting*: Processes requests at a constant rate, queuing excess requests.
- *Comparative Analysis*: Token bucket is better for LLMs because generation bursts are natural; leaky bucket is better for smoothing out traffic but may increase latency for legitimate bursty queries.

**5) Trade-off Analysis**
- **Burst Tolerance vs. Fairness**: Token buckets allow fair bursts but can be gamed by coordinated attacks. Leaky buckets ensure steady processing but may penalize legitimate users with bursty conversation patterns.

**6) How to Determine the Optimal Solution**
- For LLM inference, token bucket rate limiting is generally optimal as it matches the natural bursty nature of text generation. Combine with WAF and anomaly detection for comprehensive API security.

**7) Full Names and Explanations of Nouns and Abbreviations**
- **LLM**: Large Language Model – a language model comprising a neural network with many parameters, trained on large quantities of unlabeled text using self-supervised learning or semi-supervised training.
- **DDoS**: Distributed Denial of Service – a malicious attempt to disrupt the normal traffic of a targeted server, service, or network.
- **WAF**: Web Application Firewall – a specific type of firewall that monitors and filters HTTP traffic to and from a web application.
- **Token Bucket**: A rate limiting algorithm that allows bursts of traffic up to a certain capacity, then limits to a fixed rate.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Security & Compliance)**:
  ```mermaid
  graph TD
      A[Client Request] --> B[API Gateway TLS/Auth]
      B --> C[Data Sanitization PII Redaction]
      C --> D[Secure Model Inference]
      D --> E[Audit & Compliance Logging]
      D --> F[FinOps Cost Tracking]
  ```

- **Data Flow Diagram (Secure Inference)**:
  ```mermaid
  flowchart LR
      A[User Input] --> B[Encryption & Auth]
      B --> C[PII Detection & Redaction]
      C --> D[Model Inference Engine]
      D --> E[Output Validation]
      E --> F[Logged Audit & Metrics]
  ```
