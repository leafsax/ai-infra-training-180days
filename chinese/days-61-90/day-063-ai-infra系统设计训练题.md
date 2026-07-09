### Day 63: Tokenization 与数据集准备管道 (Tokenization and Dataset Preparation Pipeline)

**1) 题目与考察核心**：
设计一个大规模 Tokenization 和数据集准备管道，包括 Tokenizer 训练、文本分词、上下文窗口切分（Context Window Chunking）以及格式化为训练就绪的张量（Tensors）。

**2) 需求澄清与指标定义**：
- 数据量：每日处理 500 GB 清洗后的文本数据。
- 吞吐量：Tokenization 速度需达到 1000 tokens/sec per CPU core。
- 上下文窗口：目标模型上下文窗口为 128K tokens。
- 存储：Tokenized 数据以二进制格式（如 .bin 或 .tel）存储，预估大小为原始文本的 1.2 倍（因 token 编码效率）。

**3) 核心架构/技术组件设计**：
- Tokenizer 训练层：使用 SentencePiece 或 HuggingFace Tokenizers 库，基于目标语料训练 BPE（Byte-Pair Encoding）或 WordPiece Tokenizer。
- 分布式 Tokenization 层：使用 Ray 或 multiprocessing 进行并行分词，将文本转换为 token IDs。
- 上下文切分与打包层：将 tokenized 文本按 128K 窗口切分，并进行跨文档的连续性打包（Continuous packing），避免截断信息丢失。
- 存储层：将打包后的 tensor 数据保存为二进制文件，并生成索引文件（index.json）记录文档边界。

**4) 关键技术深入与可能解**：
- Tokenization 算法对比：
  - BPE (Byte-Pair Encoding)：通过迭代合并频率最高的字符对来构建词汇表，适合多语言和新词发现。
  - WordPiece：类似 BPE，但选择合并时最大化语言模型似然，常用于 BERT 系列模型。
  - SentencePiece：基于子词的分词器，将空格也视为普通字符，支持无字典分词，适合多语言。

**5) Trade-off（权衡）分析**：
- BPE vs WordPiece：BPE 更简单高效，WordPiece 在低资源语言上表现更好。
- 连续打包（Continuous Packing） vs 文档截断（Document Truncation）：连续打包能最大化利用上下文窗口，但需要复杂的状态管理；文档截断简单，但会浪费窗口容量并割裂上下文。

**6) 如何确定最优解**：
对于 128K 上下文窗口的现代 LLM，采用 SentencePiece 或 BPE Tokenizer，并结合 Continuous Packing 技术，使用 Ray 进行分布式 tokenization，确保高效利用 GPU 训练时的上下文窗口。

**7) 名词和缩写全称及解释**：
- **Tokenization**：将文本转换为 tokens（词元）的过程，tokens 是模型处理的单位。
- **Tokenizer**：执行 tokenization 的工具或模型。
- **BPE (Byte-Pair Encoding)**：字节对编码，一种数据压缩与子词分割算法。
- **WordPiece**：一种子词分词算法，常用于 BERT 模型。
- **SentencePiece**：由 Google 开发的无字典子词分词库。
- **Context Window**：模型一次处理的最大 token 数量。
- **Continuous Packing**：将多个文档的 tokens 连续打包到一个上下文窗口中，避免窗口内出现空白或截断。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 数据管道）**：
  ```mermaid
  graph TD
      A[数据源] --> B[ETL/ELT 管道 Spark/Flink]
      B --> C[数据湖 S3/Parquet]
      B --> D[向量数据库 FAISS/Weaviate]
  ```

- **数据流图（Data Flow Diagram - RAG系统）**：
  ```mermaid
  flowchart LR
      A[用户查询] --> B[RAG 编排器]
      B --> C[向量数据库检索]
      C --> D[上下文构造]
      D --> E[LLM推理]
      E --> F[最终响应]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于RAG检索与一致性**：在你的RAG架构中，如何处理文档更新时的embedding drift（嵌入漂移）或向量数据库一致性？为了满足TTFT < 200ms的SLA，检索步骤与LLM生成步骤分配的确切延迟预算是多少？
- **关于数据管道吞吐量**：你提到了使用Spark/Flink的ETL/ELT管道。如何防止数据加载器在训练期间成为瓶颈？对于会导致某些GPU工作节点饥饿的数据分布倾斜问题，你的策略是什么？
- **关于自动扩缩容的边缘情况**：对于基于自定义指标（如KV cache利用率）的扩缩容，如果指标服务器本身不可用会发生什么？如何设计回退机制以防止无限扩缩容或pod突然终止？
