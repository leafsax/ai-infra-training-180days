### Day 118: Model Deployment Registry: Routing, Canary, Blue-Green Deployments

**1) Topic and Core Examination Areas**
- Topic: Model Deployment Registry: Routing, Canary, Blue-Green Deployments.
- Core Examination Areas: Strategies for deploying model versions from the registry to serving infrastructure, including traffic routing and rollout strategies.

**2) Requirement Clarification and Metric Definitions**
- **Canary Deployment**: Releasing a new model version to a small percentage of traffic (e.g., 5%) before full rollout.
- **Blue-Green Deployment**: Running two identical production environments (Blue and Green), switching traffic from Blue to Green upon successful validation of the new version.
- **Routing Latency**: Time added by the routing layer. Target < 5ms overhead for model serving routing.

**3) Core Architecture/Technical Component Design**
- **Deployment Registry**: Tracks which model version is active in which environment (Development, Staging, Production).
- **Routing Layer**: A service or proxy (e.g., Istio, API Gateway) that routes requests to the appropriate model serving instance based on version or stage.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Istio Virtual Services**: Can route traffic between different model serving deployments based on weight or header conditions.
- **Kubernetes Deployments**: Use Kubernetes deployment objects for model serving, with labels indicating model version, enabling rolling updates or blue-green setups.

**5) Trade-off analysis**
- *Canary vs. Blue-Green*: Canary deployments allow gradual traffic increase and real-world testing but require monitoring infrastructure. Blue-green deployments allow instant rollback but require double the infrastructure during the switch.
- *Proxy Routing vs. Application-Level Routing*: Proxy routing (Istio) is transparent to the model serving code but adds infrastructure complexity. Application-level routing is simpler but requires code changes.

**6) How to determine the optimal solution**
- Use Kubernetes deployments with Istio or a similar service mesh for routing. Implement canary deployments for new model versions to validate performance and business metrics before full rollout. Use blue-green for critical models where instant rollback is required.

**7) Full names and explanations of all nouns and abbreviations**
- **API Gateway**: A server that acts as an API front-end, receiving API requests, enforcing throttling and security policies, and passing requests to the appropriate backend service.
- **Kubernetes Deployments**: A Kubernetes resource that manages the deployment of a replicated set of pods, supporting rolling updates and rollbacks.

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
