### Day 159: Supply Chain Security for AI Models and Dependencies (SBOM)

**1) Topic and Core Examination Areas**
- Topic: Supply Chain Security for AI Models and Dependencies, Software Bill of Materials (SBOM).
- Core Examination Areas: Model provenance, dependency vulnerability scanning, and securing the AI supply chain.

**2) Requirement Clarification and Metric Definitions**
- **Components**: Track all dependencies (libraries, frameworks, pre-trained models, datasets).
- **Vulnerability Metric**: < 0.1% of deployed models contain dependencies with known critical CVEs (Common Vulnerabilities and Exposures).
- **Scanning Latency**: SBOM generation and scanning must complete within 5 minutes for a model package.

**3) Core Architecture/Technical Component Design**
- **SBOM Generator**: Tooling (e.g., CycloneDX, SPDX) to create explicit lists of components used in the model and serving stack.
- **Vulnerability Scanner**: Integrates with CVE databases to scan dependencies and model formats (e.g., PyTorch, TensorFlow, Hugging Face models).
- **Model Registry with Provenance**: Stores models with cryptographic hashes and metadata about their training data and origin.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Solution A: SPDX (Software Package Data Exchange)*: Standard format for SBOMs, widely adopted.
- *Solution B: CycloneDX*: Focuses on security and supply chain component analysis, integrates well with vulnerability scanners.
- *Comparative Analysis*: SPDX is a comprehensive standard for documentation. CycloneDX is more focused on runtime security and vulnerability mapping. Both are complementary.

**5) Trade-off Analysis**
- **Comprehensiveness vs. Overhead**: Generating detailed SBOMs for AI models (including datasets and hyperparameters) adds process overhead but is critical for compliance and security auditing.

**6) How to Determine the Optimal Solution**
- Use CycloneDX for runtime vulnerability mapping and SPDX for formal documentation and compliance. Integrate SBOM generation into the CI/CD pipeline for model packaging.

**7) Full Names and Explanations of Nouns and Abbreviations**
- **SBOM**: Software Bill of Materials – a formal record containing the details and supply chain relationships of various components used in building software.
- **CVE**: Common Vulnerabilities and Exposures – a dictionary of publicly known cybersecurity vulnerabilities.
- **SPDX**: Software Package Data Exchange – an open standard format for communicating software bill of materials.
- **CycloneDX**: A lightweight SBOM standard designed for use in application security contexts and supply chain component analysis.

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


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On PII Redaction & False Positives:** For your PII detection and redaction module, what is the acceptable false-positive rate? How do you ensure the redacted text doesn't break the model's context window or cause the model to generate unsafe or nonsensical outputs?
- **On Spot Instances & Preemption:** For cost optimization using spot instances, what is the maximum acceptable preemption rate for your training jobs? How do you dynamically migrate or save checkpoints when a spot instance is terminated with only a 2-minute warning?
- **On Quantum/Post-Quantum Crypto:** You mentioned Kyber and Dilithium for post-quantum cryptography. What is the overhead in terms of latency and bandwidth when applying these algorithms to real-time inference traffic? How do you achieve crypto-agility without redesigning the serving gateway?
