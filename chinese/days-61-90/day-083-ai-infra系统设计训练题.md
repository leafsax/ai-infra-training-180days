### Day 83: AI 服务自动扩缩容基础 (Auto-Scaling Fundamentals for AI Services)

**1) 题目与考察核心**：
设计 AI 推理服务的自动扩缩容系统，考察指标驱动扩缩容与 K8s HPA。

**2) 需求澄清与指标定义**：
- 扩缩容指标：GPU 利用率 > 70% 触发扩容，< 30% 触发缩容。
- 扩缩容延迟：< 2 分钟。

**3) 核心架构/技术组件设计**：
- 监控：Prometheus 收集 GPU 指标。
- 扩缩容控制器：K8s HPA (Horizontal Pod Autoscaler) 或 KEDA。

**4) 关键技术深入与可能解**：
- 基于请求数扩缩容 vs 基于 GPU 利用率扩缩容。

**5) Trade-off（权衡）分析**：
- 请求数扩缩容响应快但可能忽略资源瓶颈；GPU 利用率更准确但延迟高。

**6) 如何确定最优解**：
采用混合指标：QPS + GPU 利用率 + 队列长度。

**7) 名词和缩写全称及解释**：
- **HPA (Horizontal Pod Autoscaler)**：Kubernetes 水平 Pod 自动扩缩容器。
- **KEDA (Kubernetes Event-Driven Autoscaling)**：基于事件的 K8s 扩缩容工具。

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
