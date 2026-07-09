### **Day 85: Canary Deployments, A/B Testing, and Shadow Deployments for Models**

**1) Topic and Core Examination Areas**
- Model deployment strategies
- A/B testing for LLMs and RAG systems
- Shadow testing and traffic splitting

**2) Requirement Clarification and Metric Definitions**
- Canary traffic percentage: Start with 5%, scale to 20%, then 100%
- A/B test duration: 7 days minimum for statistical significance
- Shadow traffic: 10% of production traffic routed to new model silently

**3) Core Architecture/Technical Component Design**
- Traffic Splitting: Istio, Kong, or API gateway for percentage-based routing
- A/B Framework: LangSmith, custom analytics, or feature flags
- Shadow Mode: Duplicate requests to new model, log outputs, but not return to user

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Canary vs A/B vs Shadow**: Canary releases gradually roll out a new version to users. A/B testing compares two versions metric-wise. Shadow routing sends traffic to a new model without affecting user experience, used for validation.
- **Feature Flags**: Enable or disable model features or versions without deploying new code.

**5) Trade-off analysis**
- Canary: Low risk, gradual rollout, but may take time to validate full performance.
- Shadow: Safe validation, but does not measure real user impact since responses are not returned.
- A/B Testing: Measures real impact, but requires statistical rigor and user segmentation.

**6) How to determine the optimal solution**
- For new model versions, start with shadow mode to validate outputs, then canary release with 5% traffic, and finally full rollout.
- For comparing model A vs model B for business metrics, use A/B testing with feature flags.

**7) Full names and explanations of nouns and abbreviations**
- **Canary Deployment**: Releasing a new version to a small subset of users before full rollout.
- **Shadow Deployment**: Routing production traffic to a new system silently to test it without affecting users.
- **Feature Flag**: A software development technique that allows features to be enabled or disabled without deploying new code.

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
