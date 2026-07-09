## 第8天：LLM推理服务系统架构

### 1) 题目与考察核心
**题目**：设计一个企业级LLM Serving系统，支持多模型、多租户、高可用。
**考察核心**：整体架构设计，包括Router、Engine、GPU集群管理。

### 2) 需求澄清与指标定义
- **并发请求**：5000 QPS。
- **多模型支持**：同时部署Llama-3-70B, Qwen-14B, Embedding模型。
- **指标**：系统可用性 99.99%，路由延迟 < 10ms。

### 3) 核心架构/技术组件设计
- **LLM Router**：负责请求分发、负载均衡、模型选择。
- **Serving Engine**：如vLLM, TGI, TensorRT-LLM。
- **Model Registry**：模型版本管理与元数据管理。

### 4) 关键技术深入与可能解
- *路由策略*：基于轮询（Round-Robin）、最少连接（Least Connections）、或基于模型容量（Capacity-based）的路由。

### 5) Trade-off（权衡）分析
- *集中式Router vs 分布式Router*：集中式Router简单但可能成为瓶颈；分布式Router（如Sidecar模式）扩展性好但增加网络跳数。

### 6) 如何确定最优解
- 采用Kubernetes + Istio进行服务网格路由，结合自定义LLM Router（如vLLM's OpenAI-compatible API server）处理模型特定逻辑。

### 7) 名词和缩写全称及解释
- **LLM Serving**：大语言模型推理服务，将训练好的模型部署为API供外部调用。
- **Router (路由)**：请求分发组件，根据策略将请求导向合适的引擎或模型。

---