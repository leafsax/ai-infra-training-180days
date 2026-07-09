### **Day 67: Document Chunking Strategies and Embedding Models**

**1) Topic and Core Examination Areas**
- Document parsing and chunking strategies (fixed-size, semantic, recursive)
- Embedding models selection and optimization
- Chunk metadata and overlap

**2) Requirement Clarification and Metric Definitions**
- Document types: PDFs, Markdown, HTML, Word docs
- Chunk size: 500-1,000 tokens per chunk
- Chunk overlap: 10-20% of chunk size
- Embedding model dimension: 768 or 1024 dimensions
- Embedding API cost: ~$0.1 per 1M tokens

**3) Core Architecture/Technical Component Design**
- Parsing Layer: pdfplumber, PyPDF, BeautifulSoup, or LlamaParse
- Chunking Layer: RecursiveCharacterTextSplitter, SemanticChunker
- Embedding Layer: OpenAI text-embedding-3-small, Cohere embed-english-v3, or open-source models (e.g., BGE-m3)
- Metadata Injection: Document source, section headers, timestamps

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Fixed-size vs Semantic Chunking**: Fixed-size (e.g., RecursiveCharacterTextSplitter) splits by delimiters (newlines, periods) and is fast but may break semantic context. Semantic chunking uses embeddings or NLP to split at logical boundaries, preserving context but requiring more compute.
- **Embedding Models**: OpenAI/Cohere models offer high accuracy and ease of use but are SaaS and cost per token. Open-source models (BGE, Sentence-Transformers) can be self-hosted, offering cost control at scale but requiring inference infrastructure.

**5) Trade-off analysis**
- Fixed-size chunking: Fast, simple, but may cut sentences or lose context.
- Semantic chunking: Preserves meaning, better retrieval quality, but slower and more complex to implement.
- SaaS vs Self-hosted embeddings: SaaS is easy and accurate but scales costly; self-hosted is cost-effective at high volume but requires GPU/CPU infrastructure and maintenance.

**6) How to determine the optimal solution**
- For general documents with mixed formats, use RecursiveCharacterTextSplitter with 500-1000 token size and 10-20% overlap.
- For complex documents (reports, papers) where context is critical, use semantic chunking or LLM-based chunking.
- For low budget and high volume, self-host open-source embeddings (BGE-m3); for highest accuracy and simplest ops, use SaaS embeddings (OpenAI, Cohere).

**7) Full names and explanations of nouns and abbreviations**
- **Token**: A piece of text (word, subword, or character) that an LLM processes.
- **Embedding**: A dense vector representation of text that captures semantic meaning.
- **BGE-m3**: BAAI General Embedding - multilingual, multi-functionality. An open-source embedding model.

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
