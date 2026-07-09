### Day 81: RAG 安全与隐私 (RAG Security and Privacy)

**1) 题目与考察核心**：
设计 RAG 系统的数据访问控制与防提示注入（Prompt Injection）机制。

**2) 需求澄清与指标定义**：
- 访问控制：基于角色的数据访问（RBAC）。
- 防注入：检测并过滤恶意查询。

**3) 核心架构/技术组件设计**：
- 权限层：在检索前验证用户权限。
- 输入过滤：使用规则或模型检测注入攻击。

**4) 关键技术深入与可能解**：
- 规则过滤 vs 模型检测。

**5) Trade-off（权衡）分析**：
- 安全性 vs 用户体验（误拦率）。

**6) 如何确定最优解**：
采用权限验证 + 输入沙箱化 + LLM 安全分类器。

**7) 名词和缩写全称及解释**：
- **RBAC (Role-Based Access Control)**：基于角色的访问控制。
- **Prompt Injection**：提示注入攻击，通过恶意输入操控 LLM 行为。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 自动扩缩容）**：
  ```mermaid
  graph TD
      A[API网关] --> B[推理服务 vLLM]
      B --> C[自定义指标服务器]
      C --> D[KEDA 扩缩容器]
      D --> E[Kubernetes 集群]
      E --> B
  ```

- **数据流图（Data Flow Diagram - 扩缩容数据流）**：
  ```mermaid
  flowchart LR
      A[请求负载] --> B[指标导出器]
      B --> C[Prometheus / DCGM]
      C --> D[KEDA 操作器]
      D --> E[扩容/缩容 Pods]
  ```
