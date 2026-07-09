### Day 117: Model Registry for Multimodal Models and Large Language Models

**1) Topic and Core Examination Areas**
- Topic: Model Registry for Multimodal Models and Large Language Models.
- Core Examination Areas: Specific considerations for registering and versioning VLMs and LLMs, including artifact size, framework dependencies, and serving configurations.

**2) Requirement Clarification and Metric Definitions**
- **Artifact Size**: LLM/VLM artifacts can be 14GB (7B model in BF16) to 140GB+ (70B model). Registry storage must handle large files.
- **Framework Dependencies**: PyTorch version, CUDA version, transformer library version (e.g., Hugging Face `transformers` 4.36+).

**3) Core Architecture/Technical Component Design**
- **Large Artifact Storage**: Integrate the registry with object storage (S3, GCS) or distributed file systems for model weights, storing only metadata and pointers in the registry database.
- **Serving Configuration Metadata**: Store inference engine (vLLM, TensorRT-LLM), parallelism settings (TP, PP), and batch size configurations in the registry.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Hugging Face Hub**: Handles large model files via Git LFS (Large File Storage) or direct S3 integration.
- **MLflow with S3 Backend**: Stores model artifacts in S3 while metadata is in a database.

**5) Trade-off analysis**
- *Registry-Stored vs. External-Stored Artifacts*: Storing artifacts in the registry database is simple but doesn't scale for large LLM files. Using external storage (S3) with registry pointers scales better but requires managing access to the storage.
- *Framework Locking vs. Flexibility*: Locking to specific framework versions ensures reproducibility but may limit serving options. Flexible registries allow multiple serving frameworks but require more metadata.

**6) How to determine the optimal solution**
- Use a model registry that supports external artifact storage (e.g., MLflow with S3, Hugging Face Hub). Store serving configurations (inference engine, parallelism, batch size) as metadata to ensure consistent deployment across environments.

**7) Full names and explanations of all nouns and abbreviations**
- **Git LFS (Large File Storage)**: A Git extension for versioning large files, such as images, videos, and machine learning models.
- **CUDA (Compute Unified Device Architecture)**: A parallel computing platform and API model created by NVIDIA, allowing software to use certain types of GPU for general purpose processing.

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
