### Day 90: 端到端 AI Infra 自动扩缩容系统设计 (End-to-End AI Infra Auto-Scaling System Design)

**1) 题目与考察核心**：
整合数据管道、RAG 系统与自动扩缩容，设计端到端 AI Infra 系统。

**2) 需求澄清与指标定义**：
- 系统 QPS：5000 QPS（检索 4000，生成 1000）。
- 端到端延迟：P99 < 3 秒。
- 可用性：99.9%。

**3) 核心架构/技术组件设计**：
- 数据管道：Kafka + Flink + 向量数据库（实时更新）。
- RAG 服务：Hybrid Search + vLLM 推理 + KEDA 扩缩容。
- 监控与告警：Prometheus + Grafana + 自动扩缩容控制器。

**4) 关键技术深入与可能解**：
- 集成挑战：数据管道延迟与 RAG 扩缩容的同步。

**5) Trade-off（权衡）分析**：
- 系统复杂度 vs 可扩展性与可靠性。

**6) 如何确定最优解**：
采用微服务架构，数据管道与 RAG 服务解耦，使用消息队列缓冲，扩缩容基于各服务独立指标，整体通过 API Gateway 统一路由与限流。

**7) 名词和缩写全称及解释**：
- **API Gateway**：API 网关，统一处理请求路由、限流与认证。
- **Microservices**：微服务架构，将系统拆分为独立部署的服务。
- **Backpressure**：背压，当处理速度跟不上流入速度时，反馈控制流入速率的机制。

--- 

以上为 Days 61-90 的 AI Infra System Design Training Question Set，涵盖数据管道、RAG系统与自动扩缩容三大主题，每日内容均包含题目与考察核心、需求澄清与指标定义、核心架构/技术组件设计、关键技术深入与可能解、Trade-off分析、如何确定最优解，以及所有出现的名词和缩写的全称及解释。

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
