## Day 28: Security and Privacy in AI Infra

### 1) Topic and Core Examination Areas
- **Topic**: Security and Privacy in AI Infrastructure.
- **Core Examination Areas**: Model theft prevention, data leakage, secure serving, and access control.

### 2) Requirement Clarification and Metric Definitions
- **Access Control Target**: RBAC (Role-Based Access Control) for model deployment and inference APIs.
- **Data Encryption**: In-transit (TLS) and at-rest encryption for model weights and user data.

### 3) Core Architecture/Technical Component Design
- **API Security**: Authentication (OAuth, API keys), rate limiting, and input validation to prevent adversarial attacks.
- **Model Encryption**: Storing model weights in encrypted form and decrypting in memory only (confidential computing).

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Confidential Computing**: Using hardware-enforced secure enclaves (e.g., NVIDIA Confidential Computing, AMD SEV) to protect model weights and data in use.
- **Watermarking**: Embedding detectable patterns in model outputs to prevent unauthorized model cloning or misuse.

### 5) Trade-off analysis
- **Security vs. Performance**: Encryption and confidential computing introduce overhead that can increase latency and reduce throughput. Evaluate the sensitivity of the model and data to determine the necessary security level.

### 6) How to determine the optimal solution
- For sensitive or proprietary models, use confidential computing and encrypted storage/transit. Implement strict RBAC and rate limiting at the API gateway. Use watermarking for models where output cloning is a risk. Balance security measures with performance SLAs.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **RBAC (Role-Based Access Control)**: A security method that restricts system access based on the roles of individual users within an organization.
- **TLS (Transport Layer Security)**: A cryptographic protocol designed to provide secure communication over a network.
- **Confidential Computing**: A security framework that uses hardware-based trusted execution environments to protect data in use.

---

