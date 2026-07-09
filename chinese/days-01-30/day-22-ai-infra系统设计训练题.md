## 第22天：推测解码（Speculative Decoding）与Early Exiting

### 1) 题目与考察核心
**题目**：设计Speculative Decoding系统以加速LLM推理生成速度。
**考察核心**：Draft Model（草稿模型），Tree Attention，验证与接受机制。

### 2) 需求澄清与指标定义
- **目标加速比**：2x - 3x 生成速度。
- **指标**：TTFT增加 < 20%，每Token延迟降低50%。

### 3) 核心架构/技术组件设计
- **Draft Model + Target Model**：使用小模型（如Llama-3-8B）生成候选Token，大模型（Llama-3-70B）进行并行验证。
- **Verification Engine**：并行计算小模型生成序列的Logits并比对。

### 4) 关键技术深入与可能解
- *Speculative Decoding*：小模型快速生成k个候选Token，大模型一次性验证这k个Token。若全部匹配，则接受k个；若部分匹配，接受匹配部分并重算。
- *Tree Attention*：扩展Speculative Decoding，支持树状结构的候选验证。

### 5) Trade-off（权衡）分析
- *Draft Model选择*：Draft模型需与Target模型架构相似以保证高接受率，但计算资源需额外分配。

### 6) 如何确定最优解
- 选择同系列但参数较小的模型作为Draft Model（如70B的Target用7B或8B的Draft），并确保两者Tokenizer一致。

### 7) 名词和缩写全称及解释
- **Speculative Decoding (推测解码)**：使用小模型生成候选序列，大模型验证以加速生成的技术。
- **Draft Model (草稿模型)**：用于生成候选Token的小模型。
- **Target Model (目标模型)**：最终验证并输出高质量结果的大模型。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 分布式训练）**：
  ```mermaid
  graph TD
      A[数据加载器] --> B[训练节点 GPU+CPU]
      B --> C[NCCL All-Reduce 网络]
      B --> D[检查点存储 S3/NFS]
      B --> E[模型注册表 MLflow]
      C --> B
  ```

- **数据流图（Data Flow Diagram - 训练流程）**：
  ```mermaid
  flowchart LR
      A[训练数据] --> B[各GPU梯度计算]
      B --> C[NCCL梯度同步]
      C --> D[权重更新]
      D --> E[检查点保存]
  ```
