### **Day 87: Metric-Driven Auto-Scaling for LLM Serving (TTFT, TPOT, P99 Latency, KV Cache utilization)**

**1) Topic and Core Examination Areas**
- LLM-specific metrics for auto-scaling
- TTFT, TPOT (Time Per Output Token), P99 latency
- KV Cache utilization and GPU memory scaling triggers

**2) Requirement Clarification and Metric Definitions**
- TTFT target: < 500 ms
- TPOT target: < 50 ms/token
- P99 latency target: < 3 seconds
- KV Cache utilization threshold: 85% of HBM

**3) Core Architecture/Technical Component Design**
- Custom Metrics: Export TTFT, TPOT, KV Cache usage from vLLM/TensorRT-LLM to Prometheus
- Auto-Scaler: KEDA or custom HPA triggered by TTFT or KV Cache utilization
- Scaling policy: Scale out when TTFT > 500 ms or KV Cache > 85%

**4) Deep Dive into Key Technologies and Possible Solutions**
- **TTFT vs TPOT Scaling**: TTFT is sensitive to request arrival and initial processing; TPOT is sensitive to batch size and GPU compute. Scaling based on TTFT prevents latency spikes; scaling based on TPOT ensures throughput.
- **KV Cache Utilization**: As KV Cache grows, GPU memory fills up, causing eviction or slowdown. Monitoring KV Cache HBM usage is a leading indicator for scaling.

**5) Trade-off analysis**
- TTFT-based scaling: Responsive to user experience, but may trigger scaling for transient spikes.
- KV Cache-based scaling: Accurate indicator of GPU memory pressure, but requires custom metrics export.

**6) How to determine the optimal solution**
- Use TTFT and P99 latency as primary scaling triggers for user-facing LLM serving.
- Use KV Cache utilization as a secondary trigger to prevent OOM or eviction issues.

**7) Full names and explanations of nouns and abbreviations**
- **TTFT**: Time to First Token.
- **TPOT**: Time Per Output Token. The average time to generate each token after the first.
- **KV Cache**: Key-Value Cache.
- **HBM**: High Bandwidth Memory.

---

