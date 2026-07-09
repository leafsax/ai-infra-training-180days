## Day 15: Multi-Model Serving and Model Mixing (MoE)

### 1) Topic and Core Examination Areas
- **Topic**: Multi-Model Serving and Mixture of Experts (MoE).
- **Core Examination Areas**: Serving multiple models on the same cluster, MoE architecture, and router mechanisms.

### 2) Requirement Clarification and Metric Definitions
- **Model Density**: In MoE, only a subset of "experts" is activated per token (e.g., 2 out of 8 experts).
- **Router Latency**: Time taken to determine which experts handle a given token. Target < 100 microseconds.

### 3) Core Architecture/Technical Component Design
- **MoE Layer**: Consists of multiple "expert" feed-forward networks and a "router" (gating network) that directs each token to the top-K experts.
- **Multi-Model Serving Engine**: A serving system that can host and route requests to different model versions or types (e.g., text, code, vision) based on request metadata.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **MoE (Mixture of Experts)**: A model architecture where each input token is processed by a selected subset of specialized sub-networks (experts). Reduces active parameter count while maintaining large total parameters.
- **Router Optimization**: Ensuring balanced load across experts to prevent stragglers (some experts becoming bottlenecks).

### 5) Trade-off analysis
- **MoE vs. Dense Models**: MoE models offer better scalability (more parameters without proportional compute cost) but introduce router complexity and potential load imbalance among experts. Dense models are simpler but scale linearly in compute and memory.

### 6) How to determine the optimal solution
- For very large models where compute cost is a constraint, use MoE architectures (e.g., Mixtral, Qwen-MoE) with optimized routers that ensure load balancing. For multi-model serving (different tasks), use a model router gateway that directs requests to the appropriate model instance based on task type.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **MoE (Mixture of Experts)**: A neural network architecture that routes each input to a specialized subset of "expert" networks, improving efficiency and scalability.
- **Expert**: A sub-network within an MoE model responsible for processing specific types of inputs.
- **Router / Gating Network**: The component in an MoE model that determines which experts should process a given token.

---

