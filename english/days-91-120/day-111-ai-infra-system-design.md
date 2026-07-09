### Day 111: Introduction to Model Registries & Model Versioning

**1) Topic and Core Examination Areas**
- Topic: Introduction to Model Registries and Model Versioning.
- Core Examination Areas: Purpose of model registries, versioning strategies, and metadata management for machine learning models.

**2) Requirement Clarification and Metric Definitions**
- **Model Version**: A unique identifier for a specific iteration of a model (e.g., `model-v1.2.0`, `model-20231015`).
- **Deployment Stages**: Development, Staging, Production. Models move through these stages in the registry.
- **Registry Uptime**: Target > 99.9% availability for model registry services to ensure CI/CD and deployment pipelines are not blocked.

**3) Core Architecture/Technical Component Design**
- **Model Registry Architecture**: A centralized service or database that stores model artifacts, metadata, versions, and deployment status.
- **Versioning Strategy**: Semantic versioning (Major.Minor.Patch) or timestamp-based versioning.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Semantic Versioning**: `MAJOR.MINOR.PATCH`. MAJOR for incompatible API changes, MINOR for backward-compatible features, PATCH for backward-compatible bug fixes.
- **Registry Types**: File-based (e.g., Hugging Face Hub), Database-backed (e.g., MLflow Model Registry), Cloud-native (e.g., AWS SageMaker Model Registry).

**5) Trade-off analysis**
- *Centralized vs. Distributed Registries*: Centralized registries provide a single source of truth but can become a bottleneck or single point of failure. Distributed registries improve availability but require consistency mechanisms.
- *Semantic vs. Timestamp Versioning*: Semantic versioning is human-readable and conveys change type. Timestamp versioning is automatic but less informative.

**6) How to determine the optimal solution**
- Use a database-backed or cloud-native model registry with semantic versioning for production ML systems. Ensure the registry supports staging environments (Development, Staging, Production) and integrates with CI/CD pipelines.

**7) Full names and explanations of all nouns and abbreviations**
- **CI/CD (Continuous Integration / Continuous Deployment)**: A software development practice where code changes are automatically tested and deployed to production.
- **Semantic Versioning (SemVer)**: A versioning scheme for software that uses a three-part format: MAJOR.MINOR.PATCH.

---

