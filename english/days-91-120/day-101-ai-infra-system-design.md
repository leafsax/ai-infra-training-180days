### Day 101: Introduction to Multimodal Training & Vision-Language Models (VLMs)

**1) Topic and Core Examination Areas**
- Topic: Introduction to Multimodal Training and Vision-Language Models (VLMs).
- Core Examination Areas: Concepts of multimodal AI, architecture of VLMs, training data requirements, and alignment techniques.

**2) Requirement Clarification and Metric Definitions**
- **QPS (Queries Per Second)**: Inference QPS for a VLM like LLaVA-7B: target 10-50 QPS depending on image resolution and context length.
- **TTFT (Time to First Token)**: For VLM inference with image input, target TTFT < 500ms for standard resolution images (e.g., 336x336 or 768x768).
- **HBM (High Bandwidth Memory)**: A 7B VLM requires ~14GB HBM for weights (BF16), plus KV cache and image encoder state. Total ~24GB per model replica on 80GB GPUs.

**3) Core Architecture/Technical Component Design**
- **VLM Architecture**: Comprises an image encoder (e.g., ViT - Vision Transformer), a projection layer (e.g., MLP or Linear), and a text LLM (e.g., Llama, Qwen).
- **Training Data Pipeline**: Images and corresponding text descriptions (captions, Q&A pairs) are processed, tokenized, and fed into the model for contrastive or generative training.

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Vision Transformer (ViT)**: Splits images into patches and processes them using transformer layers. Produces image tokens.
- **Projection Layer**: Maps image tokens to the text embedding space of the LLM. Commonly an MLP or a single linear layer.
- **Training Objectives**: Next-token prediction (generative) or contrastive learning (aligning image and text embeddings).

**5) Trade-off analysis**
- *Image Resolution vs. Compute Cost*: Higher resolution images produce more image tokens, increasing KV cache size and compute for attention. Lower resolution saves compute but may reduce visual detail understanding.
- *Separate Encoders vs. Unified Models*: Separate image and text encoders are easier to train but require a projection layer. Unified models (e.g., training ViT and LLM jointly) are more complex but may achieve better alignment.

**6) How to determine the optimal solution**
- For most VLM applications, use a pre-trained image encoder (e.g., CLIP ViT-L/14) and a pre-trained LLM, with a trainable projection layer. Train with next-token prediction on image-text pairs. Start with standard resolution (e.g., 336x336 or 768x768) and scale up only if visual detail is critical.

**7) Full names and explanations of all nouns and abbreviations**
- **VLM (Vision-Language Model)**: A model that processes and understands both visual (image/video) and textual data.
- **ViT (Vision Transformer)**: A transformer-based architecture designed for image classification and other vision tasks, processing images as sequences of patches.
- **CLIP (Contrastive Language-Image Pre-training)**: A model trained to match images and text using contrastive learning, producing aligned image and text embeddings.

---

