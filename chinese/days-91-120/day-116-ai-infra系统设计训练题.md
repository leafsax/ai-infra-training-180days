### Day 116: Model Registry Security, Access Control, and Auditing

#### 1) 题目与考察核心
设计模型注册表的安全机制，包括访问控制、审计和模型知识产权保护。

#### 2) 需求澄清与指标定义
- **访问控制粒度**：项目级、模型级、版本级权限。
- **审计日志保留**：≥ 180天。

#### 3) 核心架构/技术组件设计
- **RBAC (Role-Based Access Control)**：基于角色的访问控制。
- **审计日志服务**：记录所有模型访问、下载、部署操作。

#### 4) 关键技术深入与可能解（对比分析不同方案）
- **ABAC vs RBAC**：ABAC（基于属性）更灵活但复杂；RBAC简单常用。

#### 5) Trade-off（权衡）分析
- **灵活性 vs 管理成本**：ABAC灵活但策略管理复杂。

#### 6) 如何确定最优解
采用RBAC结合项目级隔离，满足大多数企业安全需求。

#### 7) 名词和缩写全称及解释
- **RBAC (Role-Based Access Control)**：基于角色的访问控制。
- **ABAC (Attribute-Based Access Control)**：基于属性的访问控制。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 多模态训练）**：
  ```mermaid
  graph TD
      A[多模态输入 图像/文本/音频] --> B[预处理管道]
      B --> C[训练计算 GPUs]
      C --> D[跨模态注意力层]
      C --> E[检查点与模型注册表]
  ```

- **数据流图（Data Flow Diagram - 多模态流程）**：
  ```mermaid
  flowchart LR
      A[原始多模态数据] --> B[Tokenization与编码]
      B --> C[模型前向传播]
      C --> D[损失计算]
      D --> E[反向传播与梯度同步]
  ```
