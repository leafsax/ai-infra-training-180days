## Day 6: Model Parallelism - Tensor Parallelism & Pipeline Parallelism

### 1) Topic and Core Examination Areas
- **Topic**: Model Parallelism - Tensor Parallelism (TP) and Pipeline Parallelism (PP).
- **Core Examination Areas**: Splitting model weights across GPUs (TP), splitting model layers across GPUs (PP), and hybrid DP+TP+PP strategies.

### 2) Requirement Clarification and Metric Definitions
- **Model Size**: e.g., 70B parameters, requiring >140GB in FP16, exceeding single GPU HBM.
- **TP Degree**: Number of GPUs splitting a single layer's weights (e.g., TP=4).
- **PP Degree**: Number of stages splitting the model layers (e.g., PP=4).

### 3) Core Architecture/Technical Component Design
- **Tensor Parallelism (TP)**: Splits a single layer (e.g., Linear attention layers) across multiple GPUs. Each GPU computes a part of the output, and results are aggregated.
- **Pipeline Parallelism (PP)**: Divides the model into sequential stages (layers), with each stage assigned to a different GPU. Data flows through the pipeline like an assembly line.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Megatron-LM**: A popular framework implementing TP, PP, and DP.
- **Context Parallelism / Sequence Parallelism**: Splitting the sequence dimension across GPUs to handle long contexts.
- Comparative analysis: TP minimizes communication volume per layer but requires high inter-GPU bandwidth (NVLink). PP reduces memory per GPU but introduces pipeline bubbles (idle time).

### 5) Trade-off analysis
- **TP vs. PP**: TP is efficient for layer-wise splitting but requires strong GPU-to-GPU interconnects (NVLink). PP allows scaling to more GPUs but suffers from pipeline bubbles and requires techniques like Interleaved PP or GPipe to mitigate idle time.

### 6) How to determine the optimal solution
- For models that fit in 2-4 GPUs' memory, use TP. For larger models, combine DP, TP, and PP (e.g., DP=8, TP=4, PP=4). Use interleaved pipeline parallelism to minimize pipeline bubbles.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Model Parallelism**: A distributed training strategy where different parts of a model's weights are distributed across multiple GPUs.
- **Tensor Parallelism (TP)**: A model parallelism technique that splits the tensors (weights) of a single layer across multiple GPUs.
- **Pipeline Parallelism (PP)**: A model parallelism technique that divides the model's layers into sequential stages, each assigned to a different GPU.
- **Megatron-LM**: A library by NVIDIA for training large-scale transformer models using tensor, pipeline, and data parallelism.
- **Context/Sequence Parallelism**: Techniques to distribute the processing of long input sequences across multiple GPUs.

---

