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

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 推理服务）**：
  ```mermaid
  graph TD
      A[客户端/用户] --> B[API网关 / 负载均衡器]
      B --> C[vLLM/TGI 推理引擎]
      C --> D[GPU集群 H100/A100]
      C --> E[模型权重存储 NVMe/S3]
      C --> F[KV Cache 管理器]
      D --> G[GPU监控 DCGM]
  ```

- **数据流图（Data Flow Diagram - 推理流程）**：
  ```mermaid
  flowchart LR
      A[用户提示词] --> B[API网关]
      B --> C[请求队列与批处理]
      C --> D[GPU推理计算]
      D --> E[Token生成]
      E --> F[响应流式输出]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于推理批处理与延迟**：你提到了静态批处理与连续批处理。在中途动态拆分批处理时，内核融合（kernel fusion）的确切开销是多少？当长上下文请求（如128K token）到达并迫使为较短请求进行KV cache驱逐时，如何保证TP99延迟？
- **关于分布式训练通信**：在DP/TP/PP中，如何在1024个GPU的集群中处理straggler（慢节点）节点？如果一个GPU慢5%，整个All-Reduce步骤会阻塞吗？对于使用BF16权重和梯度的NCCL All-Reduce，每步的确切通信量是多少？
- **关于内存优化**：你提到了ZeRO和PagedAttention。对于ZeRO-3，如何在优化器步骤期间最小化跨节点通信开销？对于PagedAttention，当HBM发生碎片化时，确切的驱逐策略是什么（LRU vs LFU vs 基于注意力分数）？
