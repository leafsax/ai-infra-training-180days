### **Day 81: Load Balancing for AI Model Serving (L4 vs L7, AI-specific routing)**

**1) Topic and Core Examination Areas**
- Load balancing layers: L4 (Transport) vs L7 (Application)
- AI-specific routing: model routing, tenant routing, latency-based routing
- Service mesh and load balancer options

**2) Requirement Clarification and Metric Definitions**
- Load balancer QPS handling: 50,000 QPS
- L4 vs L7 latency overhead: L4 < 1 ms, L7 1-3 ms
- Routing accuracy: > 99% for tenant-to-model routing

**3) Core Architecture/Technical Component Design**
- L4 Load Balancer: AWS ALB/NLB, HAProxy, MetalLB (Kubernetes)
- L7 Load Balancer: API Gateway, Istio, Kong, Traefik
- AI Routing: Route by model name, version, or tenant ID in the L7 layer

**4) Deep Dive into Key Technologies and Possible Solutions**
- **L4 vs L7 Load Balancing**: L4 balances based on IP and port (fast, low overhead) but cannot inspect HTTP headers or payload. L7 inspects HTTP requests, enabling routing by path, headers, or tenant ID, but adds latency.
- **AI-Specific Routing**: In multi-model serving, L7 routing is essential to direct requests to the correct model endpoint or version (canary, A/B test).

**5) Trade-off analysis**
- L4: Faster, simpler, but lacks content-based routing.
- L7: Flexible routing (by model, tenant), but higher latency and complexity.

**6) How to determine the optimal solution**
- For simple, high-throughput single-model serving, L4 load balancing is sufficient.
- For multi-model, multi-tenant, or A/B testing scenarios, use L7 load balancing with service mesh (Istio) or API gateway.

**7) Full names and explanations of nouns and abbreviations**
- **L4 Load Balancing**: Layer 4 (Transport Layer) load balancing, using IP and port.
- **L7 Load Balancing**: Layer 7 (Application Layer) load balancing, inspecting HTTP/HTTPS content.
- **ALB/NLB**: Application Load Balancer / Network Load Balancer (AWS services).
- **Istio**: An open-source service mesh for managing microservice traffic and policies.

---

