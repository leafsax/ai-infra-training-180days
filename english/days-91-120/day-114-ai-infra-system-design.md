### Day 114: Model Evaluation Metrics Registry and A/B Testing Integration

**1) Topic and Core Examination Areas**
- Topic: Model Evaluation Metrics Registry and A/B Testing Integration.
- Core Examination Areas: Storing and comparing model evaluation metrics, and integrating the registry with A/B testing frameworks for model deployment.

**2) Requirement Clarification and Metric Definitions**
- **Evaluation Metrics**: Accuracy, F1-Score, BLEU, ROUGE, latency (TTFT, TP99), throughput (TPS).
- **A/B Testing**: A method of comparing two model versions (A and B) by routing a portion of traffic to each and comparing metrics.

**3) Core Architecture/Technical Component Design**
- **Metrics Registry**: A database or service that stores evaluation metrics for each model version, enabling comparison across versions.
- **A/B Testing Gateway**: A routing layer (e.g., Istio, custom load balancer) that splits traffic between model versions based on configuration.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Istio Service Mesh**: Can route traffic between different versions of a model serving service based on weights or headers.
- **Feature Flags**: Use feature flag services (e.g., LaunchDarkly) to enable or disable model versions for specific user segments.

**5) Trade-off analysis**
- *A/B Testing vs. Canary Deployment*: A/B testing compares two versions for a specific metric (e.g., conversion rate). Canary deployment gradually rolls out a new version to all users while monitoring for errors.
- *In-Band vs. Out-of-Band Evaluation*: In-band evaluation happens during A/B testing with real traffic. Out-of-band uses offline benchmark datasets, which is faster but may not reflect real-world performance.

**6) How to determine the optimal solution**
- Store evaluation metrics from both offline benchmarks and online A/B testing in the model registry. Use canary deployments for initial rollout, followed by A/B testing to compare key business metrics before promoting a model to production.

**7) Full names and explanations of all nouns and abbreviations**
- **F1-Score**: The harmonic mean of precision and recall, offering a single metric that balances both concerns.
- **Istio**: An open-source service mesh that provides traffic management, security, and observability features.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Multimodal Training)**:
  ```mermaid
  graph TD
      A[Multimodal Inputs Image/Text/Audio] --> B[Preprocessing Pipeline]
      B --> C[Training Compute GPUs]
      C --> D[Cross-Modal Attention Layers]
      C --> E[Checkpoint & Model Registry]
  ```

- **Data Flow Diagram (Multimodal Flow)**:
  ```mermaid
  flowchart LR
      A[Raw Multimodal Data] --> B[Tokenization & Encoding]
      B --> C[Model Forward Pass]
      C --> D[Loss Computation]
      D --> E[Backward Pass & Gradient Sync]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Checkpointing & Compute Stalls:** For async checkpointing with ZeRO-3 or FSDP, how do you ensure there is no compute stall during the state snapshot phase? What is the exact RPO (Recovery Point Objective) and RTO (Recovery Time Objective) during a sudden node failure or power outage?
- **On Multimodal Alignment:** How do you align the latency of image/audio encoding with text token generation? What happens if the modalities arrive out of sync or if one modality's preprocessing takes 3x longer than the others?
- **On Model Registry & Rollbacks:** In your model registry design, how do you ensure atomic swaps between model versions during a deployment? What is the rollback mechanism if a newly registered model exhibits severe accuracy degradation or safety violations in production?
