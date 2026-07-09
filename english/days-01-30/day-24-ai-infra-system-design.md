## Day 24: Multi-Modal Model Serving (Vision-Language Models)

### 1) Topic and Core Examination Areas
- **Topic**: Multi-Modal Model Serving (Vision-Language Models, VLMs).
- **Core Examination Areas**: Serving models that process both text and images (e.g., LLaVA, GPT-4V), image preprocessing, and multimodal KV Cache.

### 2) Requirement Clarification and Metric Definitions
- **Image Resolution**: e.g., 1024x1024 pixels, affecting the number of visual tokens.
- **Visual Token Ratio**: e.g., 1 image at 1024x1024 may generate 576 visual tokens.
- **Latency Metrics**: TTFT must account for image encoding latency.

### 3) Core Architecture/Technical Component Design
- **Multimodal Serving Pipeline**: Image reception -> Image preprocessing (resize, normalize) -> Vision encoder -> Visual token generation -> Text decoder with visual context.
- **Vision Encoder Optimization**: Using optimized CNN or ViT (Vision Transformer) kernels.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Vision Encoder Caching**: Caching encoded visual representations for repeated images or similar resolutions to avoid recomputation.
- **Dynamic Batching for VLMs**: Batching requests with varying image sizes and text lengths requires careful KV Cache and compute scheduling.

### 5) Trade-off analysis
- **Pre-computing vs. On-the-fly Encoding**: Pre-computing and storing image embeddings saves compute during inference but increases storage and may not handle dynamic image modifications. On-the-fly encoding is flexible but adds to TTFT.

### 6) How to determine the optimal solution
- For VLM serving, use serving engines that support multimodal inputs (e.g., vLLM with vision support, TensorRT-LLM for VLMs). Optimize image preprocessing and vision encoder kernels to minimize TTFT.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **VLM (Vision-Language Model)**: A model capable of processing and understanding both visual (image) and textual inputs.
- **ViT (Vision Transformer)**: A transformer-based architecture designed for image classification and visual representation.
- **KV Cache in VLMs**: Includes both text and visual tokens' key and value states, requiring careful memory management due to varying visual token counts.

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
