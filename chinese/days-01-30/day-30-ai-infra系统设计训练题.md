## 第30天：端到端LLM训练与推理系统集成（MLOps）

### 1) 题目与考察核心
**题目**：设计端到端的LLM MLOps流水线，从训练、评估、量化到推理部署。
**考察核心**：CI/CD for Models, Model Registry, 自动化评估，A/B测试。

### 2) 需求澄清与指标定义
- **发布周期**：每周迭代模型版本。
- **指标**：从训练完成到推理服务上线 < 2小时，模型回滚时间 < 5分钟。

### 3) 核心架构/技术组件设计
- **Model Registry**：如MLflow, Hugging Face Hub，管理模型版本、元数据和权限。
- **CI/CD Pipeline**：自动化测试（Unit, Integration, LLM Eval）和部署。

### 4) 关键技术深入与可能解
- *LLM Evaluation*：使用自动化评测集（如MMLU, HumanEval）评估模型能力。
- *Canary Deployment & A/B Testing*：灰度发布新模型版本，对比新旧版本的业务指标（如用户停留时间、满意度）。

### 5) Trade-off（权衡）分析
- *自动化 vs 人工审核*：全自动发布速度快但风险高；引入人工审核和A/B测试增加周期但保证质量。

### 6) 如何确定最优解
- 建立基于MLflow的Model Registry，结合CI/CD进行自动化评估（Eval），通过Canary发布和监控指标（延迟、错误率、业务转化率）决定是否全量切换。

### 7) 名词和缩写全称及解释
- **MLOps (Machine Learning Operations)**：机器学习运维，将CI/CD应用于机器学习模型的生命周期管理。
- **CI/CD (Continuous Integration / Continuous Deployment)**：持续集成/持续部署。
- **Model Registry (模型注册表)**：存储、版本控制和部署模型的中心化系统。
- **Canary Deployment (金丝雀发布)**：将新版本逐步释放给小部分用户，验证稳定后再全量发布。

--- 

**总结**：
以上内容涵盖了AI Infra系统设计的30天详细训练题集，从基础指标与GPU架构，到模型并行（DP/TP/PP）、训练优化（ZeRO, FlashAttention, 量化）、推理服务（Continuous Batching, PagedAttention, 路由与扩缩容），再到MLOps与RAG Infra。每天均包含题目、指标定义、架构设计、技术深入、Trade-off分析、最优解确定及名词解释，可作为AI Infra系统设计面试与工程实践的完整参考。