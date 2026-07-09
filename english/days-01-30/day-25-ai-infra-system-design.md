## Day 25: Edge AI and On-Device Inference

### 1) Topic and Core Examination Areas
- **Topic**: Edge AI and On-Device Inference.
- **Core Examination Areas**: Deploying LLMs or smaller models on edge devices (phones, IoT, edge servers), quantization, and latency/privacy constraints.

### 2) Requirement Clarification and Metric Definitions
- **Device Memory**: e.g., 8GB-12GB RAM on edge servers or mobile devices.
- **Latency Target**: < 500ms TTFT for on-device interactive applications.
- **Power Constraints**: TDP (Thermal Design Power) limits for edge devices.

### 3) Core Architecture/Technical Component Design
- **On-Device Inference Engines**: e.g., Apple CoreML, Qualcomm QNN, NVIDIA Jetson TensorRT, ONNX Runtime.
- **Model Compression**: Heavy use of quantization (INT4/INT8), pruning, and distillation to fit models on edge devices.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **ONNX (Open Neural Network Exchange)**: An open format for representing machine learning models, enabling deployment across various edge inference runtimes.
- **Model Distillation**: Training a smaller "student" model to mimic the behavior of a larger "teacher" model, reducing size and latency.

### 5) Trade-off analysis
- **Accuracy vs. Efficiency**: Aggressive quantization and distillation improve on-device latency and memory usage but may reduce model accuracy or capability. Choose the compression level that meets the application's accuracy SLA.

### 6) How to determine the optimal solution
- For edge or on-device inference, use quantized models (INT8/INT4) via runtimes like ONNX Runtime, TensorRT-LLM (for edge GPUs), or platform-specific tools (CoreML, QNN). Use distillation if model capability must be preserved while reducing size.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Edge AI**: Deploying AI models on local devices (edge servers, smartphones, IoT) rather than in centralized cloud datacenters.
- **ONNX (Open Neural Network Exchange)**: An open format for building and deploying machine learning models across various frameworks and hardware.
- **TDP (Thermal Design Power)**: The maximum amount of heat a component generates that the cooling system must dissipate, often used to define power limits for devices.

---



### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram (Distributed Training)**:
  ```mermaid
  graph TD
      A[Data Loader] --> B[Training Nodes GPU+CPU]
      B --> C[NCCL All-Reduce Network]
      B --> D[Checkpoint Storage S3/NFS]
      B --> E[Model Registry MLflow]
      C --> B
  ```

- **Data Flow Diagram (Training)**:
  ```mermaid
  flowchart LR
      A[Training Data] --> B[Gradient Compute per GPU]
      B --> C[NCCL Gradient Sync]
      C --> D[Weight Update]
      D --> E[Checkpoint Save]
  ```


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Inference Batching & Latency:** You mentioned Static vs. Continuous Batching. What is the exact kernel fusion overhead when dynamically splitting batches mid-generation? How do you guarantee TP99 latency when a long-context request (e.g., 128K tokens) arrives and forces KV cache eviction for shorter requests?
- **On Distributed Training Communication:** In DP/TP/PP, how do you handle straggler nodes in a 1024-GPU cluster? If one GPU is 5% slower, does the entire All-Reduce step block? What is the exact communication volume per step for NCCL All-Reduce with BF16 weights and gradients?
- **On Memory Optimization:** You cited ZeRO and PagedAttention. For ZeRO-3, how do you minimize the cross-node communication overhead during the optimizer step? For PagedAttention, what is the exact eviction policy (LRU vs. LFU vs. Attention-score-based) when HBM fragmentation occurs?
