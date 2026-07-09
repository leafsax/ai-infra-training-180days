## Day 58: A/B Testing and Canary Deployments for LLM Services

### 1) Topic and Core Examination Areas
**Topic**: Deployment Strategies for LLM Serving.
**Core Examination Areas**: Canary releases, A/B testing, and traffic routing for model updates.

### 2) Requirement Clarification and Metric Definitions
- **Canary Deployment**: Releasing a new model version to a small percentage of traffic (e.g., 5%) before full rollout.
- **A/B Testing**: Comparing two model versions (or prompts) by splitting traffic and measuring metrics like user engagement or accuracy.
- **Rollback Time**: Time to revert to the previous model version if the new one performs poorly.

### 3) Core Architecture/Technical Component Design
- **Feature Flag / Routing Layer**: API gateway or service mesh that routes traffic to different model versions based on rules or percentages.
- **Metrics Comparison Dashboard**: Compares TTFT, TPS, and business metrics between versions.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Canary Releases**: Minimize risk by exposing the new model to a small user segment. Monitor for errors or latency degradation.
- **Shadow Deployments**: Route live traffic to both old and new models, but only use the new model's outputs for testing, not for user response.

### 5) Trade-off Analysis
- **Big Bang Release**: Fast rollout, but high risk if the new model has issues.
- **Canary/Shadow Deployment**: Lower risk, allows validation, but requires infrastructure to route and compare traffic.

### 6) How to Determine the Optimal Solution
For production LLM serving, canary deployments with shadow testing and metrics comparison are optimal to ensure new model versions do not degrade user experience.

### 7) Glossary: Full Names and Explanations
- **Canary Deployment**: A release strategy where a new version of a service or model is deployed to a small subset of users or traffic before a full rollout.
- **A/B Testing**: An experiment where two versions (A and B) are compared by splitting traffic to measure which performs better against specific metrics.
- **Shadow Deployment**: A testing technique where live traffic is sent to a new model version in the background, but the user only receives the response from the stable version.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (A/B Testing & Canary Deployments)**:
  ```mermaid
  graph TD
      A[User Traffic] --> B[API Gateway / Service Mesh]
      B --> C[Canary Router / Feature Flags]
      C --> D[Stable Model Version]
      C --> E[Canary Model Version]
      D --> F[Metrics & Logging Dashboard]
      E --> F
  ```

- **Data Flow Diagram (Canary Release Flow)**:
  ```mermaid
  flowchart LR
      A[Incoming Request] --> B[Traffic Splitter]
      B -->|95%| C[Stable Model]
      B -->|5%| D[Canary Model]
      C --> E[User Response]
      D --> F[Shadow Metrics Collection]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Traffic Splitting & Routing:** You mentioned feature flags and service mesh for traffic routing. What is the exact algorithm for percentage-based traffic splitting? How do you ensure sticky sessions or user-specific routing during a canary release without introducing significant latency overhead?
- **On Statistical Significance & A/B Testing:** For A/B testing two model versions, how do you determine the minimum sample size required to achieve statistical significance? What metrics do you track to ensure the canary model does not degrade user engagement or increase error rates?
- **On Rollback & Shadow Deployments:** In a shadow deployment, you route live traffic to both old and new models. How do you handle stateful operations or side-effects (e.g., logging, analytics) that should only be triggered by the stable model? What is the exact RTO (Recovery Time Objective) for a zero-downtime rollback if the canary model exhibits severe issues?
