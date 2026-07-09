### Day 78: 多模态 RAG 系统 (Multi-Modal RAG Systems)

**1) 题目与考察核心**：
设计支持图像、表格、文档的多模态 RAG。

**2) 需求澄清与指标定义**：
- 多模态检索命中率 > 80%。
- 延迟：多模态检索 P99 < 500 ms。

**3) 核心架构/技术组件设计**：
- 图像/表格解析：VL（Vision-Language）模型提取描述。
- 混合存储：向量库 + 图数据库 + 对象存储。

**4) 关键技术深入与可能解**：
- 表格解析：HTML 解析 vs VL 模型提取。

**5) Trade-off（权衡）分析**：
- VL 模型精度高但延迟与成本高。

**6) 如何确定最优解**：
对表格使用结构化解析（如 Pandas AI），对图像使用 CLIP/VLM 嵌入。

**7) 名词和缩写全称及解释**：
- **VLM (Vision-Language Model)**：视觉-语言模型。

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
