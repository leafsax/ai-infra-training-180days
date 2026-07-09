### Day 41: 推理服务的自动扩缩容：HPA, VPA, 预测性扩缩容

**1) 题目与考察核心**
设计支持流量波动的推理服务自动扩缩容系统。考察核心：HPA、VPA与预测性扩缩容机制。

**2) 需求澄清与指标定义**
- **流量特征**：工作日峰值QPS 5000，夜间低谷QPS 500。
- **扩缩容目标**：P99延迟 < 3秒，资源成本优化。

**3) 核心架构/技术组件设计**
- 监控层：Prometheus + GPU Metrics（nvml, dcgm）。
- 扩缩容控制器：K8s HPA（Horizontal Pod Autoscaler），结合自定义指标（如GPU利用率、队列长度）。

**4) 关键技术深入与可能解**
- **HPA (Horizontal Pod Autoscaler)**：基于CPU/GPU利用率或自定义指标增加/减少Pod数量。
- **VPA (Vertical Pod Autoscaler)**：调整单个Pod的CPU/GPU资源请求。
- **预测性扩缩容（Predictive Scaling）**：基于历史流量时序模型（如Prophet、LSTM）提前扩容，应对突发流量。

**5) Trade-off（权衡）分析**
- HPA响应快但可能因扩容延迟导致P99延迟超标；预测性扩缩容提前准备资源，但模型不准可能导致资源浪费。

**6) 如何确定最优解**
结合HPA（基于实时GPU利用率与请求队列长度）与预测性扩缩容（基于昼夜流量模式），设置扩容冷却期（Cooldown）避免抖动。

**7) 名词和缩写全称及解释**
- **HPA (Horizontal Pod Autoscaler)**：K8s横向自动扩缩容组件。
- **VPA (Vertical Pod Autoscaler)**：K8s纵向自动扩缩容组件，调整资源请求大小。
- **Prometheus / DCGM**：开源监控系统与NVIDIA数据中心GPU管理工具。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 高级推理服务）**：
  ```mermaid
  graph TD
      A[负载均衡器] --> B[推理 Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU资源管理器]
      D --> E[Kubernetes / Karpenter]
      B --> F[遥测 DCGM/OpenTelemetry]
  ```

- **数据流图（Data Flow Diagram - 连续批处理）**：
  ```mermaid
  flowchart LR
      A[ incoming 请求] --> B[连续批处理调度器]
      B --> C[KV Cache 查找与更新]
      C --> D[GPU Tensor Core 计算]
      D --> E[输出 Tokens]
      E --> F[流式响应]
  ```
