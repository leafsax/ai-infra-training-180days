### **Day 74: Multi-modal RAG (Images, Videos, Documents, PDFs)**

**1) Topic and Core Examination Areas**
- Multi-modal document parsing (PDFs with images, tables, charts)
- Multi-modal embedding and retrieval (CLIP, image embeddings)
- Multi-modal LLM generation (GPT-4V, Claude 3, LLaVA)

**2) Requirement Clarification and Metric Definitions**
- Document types: PDFs with images, tables, OCR text
- Image embedding dimension: 512 or 768 (CLIP embeddings)
- Multi-modal retrieval QPS: 2,000 QPS
- Multi-modal LLM inference latency: TTFT < 1 second, total < 5 seconds

**3) Core Architecture/Technical Component Design**
- Parsing Layer: PDF extraction with layout analysis (LayoutLM, DocParse), OCR for images (Tesseract, AWS Textract)
- Multi-modal Embedding: CLIP (Contrastive Language-Image Pre-training) for image-text pairs, or dedicated image embeddings
- Storage: Vector DB for text and image embeddings, with metadata linking images to text chunks
- Generator: Multi-modal LLM (GPT-4V, Claude 3 Sonnet/Opus, LLaVA)

**4) Deep Dive into Key Technologies and Possible Solutions**
- **OCR vs Layout-aware parsing**: OCR extracts text from images but may lose structure. Layout-aware models (LayoutLM) preserve document structure (headers, tables, columns), improving chunking quality.
- **CLIP vs Dedicated Image Embeddings**: CLIP provides joint image-text embeddings, enabling cross-modal search. Dedicated image embeddings (e.g., ResNet features) are optimized for visual similarity but not cross-modal semantic search.

**5) Trade-off analysis**
- OCR vs Layout-aware: Layout-aware is more accurate for structured documents but requires more compute and specialized models.
- Multi-modal LLMs: GPT-4V/Claude 3 offer high quality but are SaaS and expensive; open-source multi-modal models (LLaVA) are cheaper to run but may have lower accuracy or latency.

**6) How to determine the optimal solution**
- For PDFs with complex layouts and tables, use layout-aware parsing + OCR.
- For cross-modal search (image-text), use CLIP embeddings.
- For high-quality multi-modal generation, use GPT-4V or Claude 3; for cost-effective self-hosted, consider LLaVA or Qwen-VL.

**7) Full names and explanations of nouns and abbreviations**
- **OCR**: Optical Character Recognition. Technology that converts images of text into machine-encoded text.
- **CLIP**: Contrastive Language-Image Pre-training. A model that learns joint representations of images and text.
- **LLaVA**: Large Language-and-Vision Assistant. An open-source multi-modal LLM.
- **GPT-4V**: GPT-4 with Vision capabilities.

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
