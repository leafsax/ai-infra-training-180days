### **Day 61: Data Ingestion and ETL/ELT Pipelines for ML Models**

**1) Topic and Core Examination Areas**
- Data ingestion patterns (batch vs streaming)
- ETL (Extract, Transform, Load) vs ELT (Extract, Load, Transform) pipelines
- Data pipeline orchestration and reliability

**2) Requirement Clarification and Metric Definitions**
- Data ingestion volume: 10 TB/day of structured and unstructured data
- QPS (Queries Per Second) of ingestion endpoints: ~10,000 QPS for real-time event streams
- Throughput (TPS - Transactions Per Second): 5,000 TPS for batch ETL jobs
- Latency metrics: Batch ETL SLA: T+1 day (24 hours); Stream processing latency: P99 < 2 seconds
- Storage capacity: Data Lake (e.g., AWS S3) requires 100 TB initial, growing at 1 TB/week

**3) Core Architecture/Technical Component Design**
- Data Sources: APIs, databases (PostgreSQL, MongoDB), event logs
- Ingestion Layer: Apache Kafka or AWS Kinesis for stream data; AWS DMS or Fivetran for database sync
- Processing Layer: Apache Spark (batch), Apache Flink (stream)
- Storage Layer: Data Lake (S3, Parquet format), Data Warehouse (Snowflake, BigQuery)
- Orchestration: Apache Airflow or Prefect

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Batch ETL vs Stream ETL**: Batch ETL processes data in large chunks at scheduled intervals (e.g., nightly). Stream ETL processes data continuously as it arrives.
- **ETL vs ELT**: ETL transforms data before loading into the target warehouse; ELT loads raw data first, then transforms inside the warehouse using its compute power.
- **Orchestration Tools**: Apache Airflow uses DAGs (Directed Acyclic Graphs) and is Python-centric; Prefect offers modern workflow orchestration with better error handling and dynamic task generation.

**5) Trade-off analysis**
- ETL vs ELT: ETL requires more compute in the transformation layer and is better for strict data governance before storage; ELT leverages modern DW compute, is faster to implement, but raw data is stored first.
- Batch vs Stream: Batch is cost-effective, easier to debug, and guarantees exactly-once processing; Stream provides low latency but is complex to manage (state management, watermarking).

**6) How to determine the optimal solution**
- If business requires real-time dashboards or real-time ML features (e.g., fraud detection), choose Stream ETL + Kafka + Flink.
- If business is analytics-focused with T+1 requirements, choose ELT + Spark + Airflow + Data Warehouse.

**7) Full names and explanations of nouns and abbreviations**
- **ETL**: Extract, Transform, Load. A process to collect data, transform it to fit a target schema, and load it into a destination.
- **ELT**: Extract, Load, Transform. Similar to ETL but load happens before transformation.
- **QPS**: Queries Per Second. A measure of the number of queries or requests a system can handle per second.
- **TPS**: Transactions Per Second. A measure of the number of transactions a system processes per second.
- **P99 Latency**: The 99th percentile latency, meaning 99% of requests are faster than this value.
- **DAG**: Directed Acyclic Graph. A graph with nodes and directed edges that has no cycles, used to represent workflow dependencies.

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
