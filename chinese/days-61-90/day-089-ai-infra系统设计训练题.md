### Day 89: 成本优化的自动扩缩容 (Cost-Optimized Auto-Scaling)

**1) 题目与考察核心**：
设计成本优化的扩缩容策略，考察 Spot 实例、右-sizing（Right-sizing）与混合精度。

**2) 需求澄清与指标定义**：
- 成本目标：降低 30% 推理成本。
- 实例类型：On-Demand + Spot 实例混合。

**3) 核心架构/技术组件设计**：
- 混合实例调度：K8s node selectors 区分 Spot/On-Demand。
- 右-sizing：根据实际 HBM 与 GPU Util 调整实例规格。

**4) 关键技术深入与可能解**：
- Spot 实例 vs On-Demand：Spot 成本低但可能被回收（Preempted）。

**5) Trade-off（权衡）分析**：
- 成本节省 vs 服务可用性。

**6) 如何确定最优解**：
对无状态推理服务使用 Spot 实例，结合自动重试与副本冗余；对关键服务使用 On-Demand。

**7) 名词和缩写全称及解释**：
- **Spot Instances**：竞价实例，云提供商的闲置容量，成本低但不稳定。
- **Right-sizing**：根据实际工作负载调整资源规格，避免过度配置。

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
