### **Day 71: Agentic RAG and Dynamic Retrieval (ReAct, Tool Use)**

**1) Topic and Core Examination Areas**
- Agentic RAG architectures
- ReAct (Reasoning and Acting) pattern
- Dynamic retrieval and tool use

**2) Requirement Clarification and Metric Definitions**
- Agent decision latency: P99 < 2 seconds per reasoning step
- Number of tool calls per query: Average 2-3, max 5
- Success rate of tool use: > 90%

**3) Core Architecture/Technical Component Design**
- Agent Framework: LangChain Agents, LlamaIndex Agents, or AutoGen
- Reasoning Loop: ReAct pattern (Thought, Action, Observation)
- Tools: Search API, Vector DB retriever, SQL database, custom APIs

**4) Deep Dive into Key Technologies and Possible Solutions**
- **ReAct vs Prompt-only RAG**: ReAct enables the LLM to reason and take actions (tool calls) iteratively, suitable for complex queries. Prompt-only RAG is a single retrieval + generation step, simpler but less capable for multi-step reasoning.
- **LangChain Agents vs LlamaIndex Agents**: LangChain agents are highly flexible with many tool integrations but can be verbose. LlamaIndex agents are optimized for data-centric RAG and tool use over datasets.

**5) Trade-off analysis**
- Agentic RAG: More capable for complex, multi-step queries, but introduces latency, potential for infinite loops, and higher cost (more LLM calls).
- ReAct: Structured reasoning, but requires careful tool design and error handling.

**6) How to determine the optimal solution**
- For simple, document-based Q&A, use standard RAG (single retrieval + generation).
- For queries requiring external data, multi-step reasoning, or tool use, implement Agentic RAG with ReAct pattern.
- Set max tool calls and timeout to prevent infinite loops and control costs.

**7) Full names and explanations of nouns and abbreviations**
- **ReAct**: Reasoning and Acting. A prompting strategy that combines reasoning (chain-of-thought) and action (tool use) for LLM agents.
- **Agentic RAG**: RAG systems augmented with agentic capabilities (planning, tool use, reflection).

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
