### Day 68: 数据安全与合规管道 (Data Security and Compliance Pipeline)

**1) 题目与考察核心**：
设计包含 PII（个人身份信息）检测、数据脱敏与 GDPR 合规的数据处理管道。

**2) 需求澄清与指标定义**：
- PII 检测准确率：> 98%。
- 处理延迟：T+1 批处理，或实时流处理延迟 < 3 秒。
- 合规要求：支持 GDPR、CCPA 数据删除请求（Right to be Forgotten）。

**3) 核心架构/技术组件设计**：
- PII 检测层：使用 NER（命名实体识别）模型或规则引擎检测姓名、邮箱、身份证号等。
- 脱敏层：使用替换、加密或泛化技术脱敏 PII 数据。
- 合规管理：记录数据访问日志，支持数据删除与匿名化请求。

**4) 关键技术深入与可能解**：
- NER-based PII 检测 vs 规则引擎：NER 模型（如 spaCy, BERT-NER）泛化能力强，但需标注数据；规则引擎（正则表达式）准确但维护成本高。

**5) Trade-off（权衡）分析**：
- 精确脱敏 vs 数据效用：过度脱敏会降低数据质量，影响模型训练；需平衡隐私与效用。

**6) 如何确定最优解**：
采用混合方法：规则引擎处理已知 PII 模式，NER 模型处理未预见实体，结合人工审核流程处理高置信度告警。

**7) 名词和缩写全称及解释**：
- **PII (Personally Identifiable Information)**：个人身份信息，如姓名、身份证号、邮箱等。
- **GDPR (General Data Protection Regulation)**：欧盟通用数据保护条例。
- **NER (Named Entity Recognition)**：命名实体识别，NLP 任务中识别文本中的实体（如人名、地名）。
- **CCPA (California Consumer Privacy Act)**：加州消费者隐私法案。

---

