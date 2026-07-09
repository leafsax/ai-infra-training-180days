### Day 159: AI系统可审计性与可解释性 (Auditability & Explainability in AI Systems)

**1) 题目与考察核心**
设计具备可解释性和可审计能力的AI推理服务。考察核心：XAI（可解释AI）技术、决策日志、审计追踪。

**2) 需求澄清与指标定义**
- **业务场景**：信贷审批AI服务，需向用户和监管机构解释拒贷原因。
- **解释生成延迟**：< 100ms，不影响主推理TP99（< 500ms）。
- **审计日志完整性**：100%记录输入、输出、模型版本、特征重要性。

**3) 核心架构/技术组件设计**
- 可解释性引擎：集成SHAP或LIME技术，生成特征重要性解释。
- 审计日志模块：结构化记录所有决策相关元数据。
- 解释展示API：向用户或审计员提供解释结果。

**4) 关键技术深入与可能解**
- **SHAP (Shapley Additive exPlanations)** vs **LIME (Local Interpretable Model-agnostic Explanations)**：SHAP提供理论保证的全局和局部解释，计算成本高；LIME局部近似，速度快但缺乏全局一致性。
- **内置解释模型** vs **后处理解释工具**：内置（如可解释模型Tree）解释自然但表达能力有限；后处理工具（如SHAP）适用于黑盒模型但增加延迟。

**5) Trade-off（权衡）分析**
- 解释准确性 vs 计算开销：SHAP准确但慢，LIME快但局部近似。
- 黑盒模型性能 vs 可解释性：高性能大模型通常不可解释，简单模型可解释但性能低。

**6) 如何确定最优解**
信贷场景采用后处理SHAP解释（异步生成或缓存），主推理同步返回决策，解释结果在100ms内通过缓存或快速近似SHAP提供。

**7) 名词和缩写解释**
- **XAI (Explainable AI)**：可解释人工智能，使AI决策过程可理解的技术。
- **SHAP (Shapley Additive exPlanations)**：基于博弈论的特征重要性解释方法。
- **LIME (Local Interpretable Model-agnostic Explanations)**：通过局部可解释模型近似黑盒模型输出的技术。

---

