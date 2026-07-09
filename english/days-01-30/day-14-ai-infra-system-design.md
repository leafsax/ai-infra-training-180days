## Day 14: Routing and Load Balancing in LLM Serving

### 1) Topic and Core Examination Areas
- **Topic**: Request Routing and Load Balancing for LLM Serving.
- **Core Examination Areas**: API gateways, request distribution across model replicas, and affinity routing.

### 2) Requirement Clarification and Metric Definitions
- **Availability Target**: 99.9% uptime for serving endpoints.
- **Routing Latency**: < 5ms for gateway-level request routing.

### 3) Core Architecture/Technical Component Design
- **API Gateway / Load Balancer**: Distributes incoming HTTP/gRPC requests to available serving engine instances.
- **Model Replicas**: Multiple instances of the same model deployed for scaling and fault tolerance.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Round-Robin vs. Least-Connections vs. Latency-Based Routing**: 
  - *Round-Robin*: Simple, distributes requests evenly.
  - *Least-Connections/Queue*: Routes to the instance with the fewest active requests or shortest queue.
  - *Affinity Routing*: Routes requests from the same user or session to the same replica to share KV Cache state (if applicable).

### 5) Trade-off analysis
- **Simplicity vs. Optimization**: Round-robin is simple but may lead to imbalanced loads if request lengths vary significantly. Latency-based or queue-length-based routing optimizes for lower TTFT but requires more complex monitoring and routing logic.

### 6) How to determine the optimal solution
- For LLM serving with variable request lengths, use queue-length or latency-based routing at the API gateway or within the serving engine's distributed frontend to balance the load and minimize TTFT.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Load Balancing**: Distributing network or application traffic across multiple servers or instances to ensure no single instance is overwhelmed.
- **API Gateway**: A service that acts as an entry point for a system, handling requests, authentication, routing, and rate limiting.
- **Affinity Routing**: A routing strategy that ensures requests from the same client or session are directed to the same server instance.

---

