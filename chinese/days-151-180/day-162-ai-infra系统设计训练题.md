### Day 162: 模型量化与剪枝 (Model Quantization & Pruning)

**1) 题目与考察核心**
通过模型压缩技术降低AI推理成本。考察核心：量化（Quantization）、剪枝（Pruning）、成本-精度权衡。

**2) 需求澄清与指标定义**
- **业务场景**：将70B参数模型部署到成本受限的推理集群。
- **显存优化目标**：模型显存占用从 140GB (FP16) 降至 ≤ 40GB。
- **准确率下降**：≤ 2%。
- **延迟指标**：TP99 < 300ms，吞吐量提升 ≥ 2x。

**3) 核心架构/技术组件设计**
- 模型转换管道：FP16 -> INT8/INT4 量化或剪枝。
- 量化感知训练 (QAT) 或后训练量化 (PTQ) 模块。
- 推理引擎适配：支持INT4/INT8的推理引擎（如vLLM, TensorRT-LLM）。

**4) 关键技术深入与可能解**
- **PTQ (Post-Training Quantization)** vs **QAT (Quantization-Aware Training)**：PTQ快速但精度损失大；QAT训练阶段模拟量化，精度保留好但需重新训练或微调。
- **权重量化** vs **激活量化**：权重量化容易实现（静态）；激活量化动态范围大，需 per-tensor 或 per-token 量化策略。

**5) Trade-off（权衡）分析**
- 压缩率 vs 精度：更高压缩率（如INT4）导致更大精度下降。
- 训练成本 vs 部署成本：QAT增加训练成本，但降低部署显存和延迟成本。

**6) 如何确定最优解**
采用PTQ + per-channel INT8量化 + vLLM引擎，若精度下降>2%，则采用QAT微调，平衡成本与精度。

**7) 名词和缩写解释**
- **Quantization (量化)**：将模型权重/激活从高精度（FP16/FP32）转换为低精度（INT8/INT4）的技术。
- **Pruning (剪枝)**：移除模型中不重要的权重或神经元，减少模型大小。
- **PTQ (Post-Training Quantization)**：后训练量化，训练完成后直接量化模型。
- **QAT (Quantization-Aware Training)**：量化感知训练，在训练过程中模拟量化误差以保留精度。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 安全与合规）**：
  ```mermaid
  graph TD
      A[客户端请求] --> B[API网关 TLS/认证]
      B --> C[数据清洗 PII脱敏]
      C --> D[安全模型推理]
      D --> E[审计与合规日志]
      D --> F[FinOps成本追踪]
  ```

- **数据流图（Data Flow Diagram - 安全推理）**：
  ```mermaid
  flowchart LR
      A[用户输入] --> B[加密与认证]
      B --> C[PII检测与脱敏]
      C --> D[模型推理引擎]
      D --> E[输出验证]
      E --> F[记录审计与指标]
  ```
