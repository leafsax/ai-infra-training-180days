### **Day 62: Data Versioning, Lineage, and Datasets (DVC, MLflow)**

**1) Topic and Core Examination Areas**
- Data versioning control (similar to Git for code)
- Data lineage and traceability
- Experiment tracking and dataset management

**2) Requirement Clarification and Metric Definitions**
- Dataset size: 500 GB of training data, versioned weekly
- Number of experiments: 200+ model training experiments per month
- Lineage tracking: Must trace model predictions back to the exact dataset version and feature configuration
- Storage metric: Versioned data storage cost ~$50/TB/month

**3) Core Architecture/Technical Component Design**
- Versioning Layer: DVC (Data Version Control) for data, models, and metrics integration with Git
- Experiment Tracking: MLflow or Weights & Biases (W&B)
- Storage: S3 or GCS for data and model artifacts; Git for code and DVC metafiles
- Lineage Tool: Apache Atlas or MLflow Projects

**4) Deep Dive into Key Technologies and Possible Solutions**
- **DVC vs Git LFS**: Git LFS stores large files directly in the Git repository, which can bloat the repo. DVC stores data in remote storage (S3/GCS) and keeps only metafiles (`.dvc`) in Git, enabling efficient versioning.
- **MLflow vs W&B**: MLflow is open-source, modular (Tracking, Projects, Models, Registry), and self-hostable. W&B is a SaaS-first platform with rich visualization and collaboration features but can be costlier at scale.

**5) Trade-off analysis**
- DVC: Lightweight, integrates with Git, but requires custom pipeline for complex lineage.
- MLflow: Comprehensive experiment tracking and model registry, but can be complex to set up for full data lineage.
- W&B: Excellent UX and real-time collaboration, but vendor lock-in and higher cost for large teams.

**6) How to determine the optimal solution**
- For open-source, self-hosted, and Git-integrated data versioning, choose DVC + S3.
- For comprehensive experiment tracking and model registry with a team, choose MLflow.
- For SaaS-first, rich visualization, and AI research teams, choose W&B.

**7) Full names and explanations of nouns and abbreviations**
- **DVC**: Data Version Control. An open-source version control system for machine learning projects.
- **Git LFS**: Git Large File Storage. An extension for Git to handle large files.
- **MLflow**: An open-source platform to manage the ML lifecycle (experiment tracking, packaging, deployment).
- **W&B**: Weights & Biases. A SaaS platform for experiment tracking, visualization, and collaboration.
- **S3**: Amazon Simple Storage Service. Object storage service offered by AWS.
- **GCS**: Google Cloud Storage. Object storage service offered by Google Cloud.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Data Pipeline)**:
  ```mermaid
  graph TD
      A[Data Sources] --> B[ETL/ELT Pipeline Spark/Flink]
      B --> C[Data Lake S3/Parquet]
      B --> D[Vector Database FAISS/Weaviate]
  ```

- **Data Flow Diagram (RAG System)**:
  ```mermaid
  flowchart LR
      A[User Query] --> B[RAG Orchestrator]
      B --> C[Vector DB Retrieval]
      C --> D[Context Construction]
      D --> E[LLM Inference]
      E --> F[Final Response]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On RAG Retrieval & Consistency:** In your RAG architecture, how do you handle embedding drift or vector database consistency during document updates? What is the exact latency budget allocated for the retrieval step vs. the LLM generation step to meet a TTFT < 200ms SLA?
- **On Data Pipeline Throughput:** You mentioned ETL/ELT pipelines with Spark/Flink. How do you prevent the data loader from becoming the bottleneck during training? What is the strategy for handling skewed data distributions that cause certain GPU workers to starve?
- **On Auto-Scaling Edge Cases:** For custom metrics-based scaling (e.g., KV cache utilization), what happens if the metrics server itself becomes unavailable? How do you design a fallback mechanism to prevent infinite scaling or sudden pod termination?
