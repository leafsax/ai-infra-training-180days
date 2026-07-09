## 第24天：服务扩展性与自动扩缩容（Auto-scaling）

### 1) 题目与考察核心
**题目**：设计LLM Serving集群的自动扩缩容系统，应对流量峰值。
**考察核心**：KEDA, HPA (Horizontal Pod Autoscaler), 基于Queues/Custom Metrics的扩缩容。

### 2) 需求澄清与指标定义
- **流量特征**：工作日高峰QPS 5000，低谷QPS 500。
- **指标**：扩缩容响应时间 < 2分钟，P99延迟在扩容期间不恶化。

### 3) 核心架构/技术组件设计
- **KEDA (Kubernetes Event-driven Autoscaling)**：基于请求队列长度或自定义指标（如vLLM's `num_requests_waiting`）触发扩缩容。
- **Model Warm-up**：新Pod启动后预加载模型权重再进行流量接入。

### 4) 关键技术深入与可能解
- *CPU/Memory HPA vs Custom Metrics*：传统HPA基于CPU使用率，但LLM服务中CPU使用率低而GPU排队长，需基于Custom Metrics（如排队请求数）。

### 5) Trade-off（权衡）分析
- *快速扩容 vs 成本*：快速扩容保证SLA但可能产生闲置GPU成本；保守扩容降低成本但可能触发延迟SLA违规。

### 6) 如何确定最优解
- 使用KEDA基于`num_requests_waiting`或`queue_time`指标进行HPA，并设置合理的Cooldown period（如5分钟）防止震荡。

### 7) 名词和缩写全称及解释
- **HPA (Horizontal Pod Autoscaler)**：Kubernetes的水平Pod自动扩缩容控制器。
- **KEDA (Kubernetes Event-driven Autoscaling)**：基于事件驱动的Kubernetes扩缩容组件。
- **Cooldown Period**：扩缩容操作后的冷却时间，防止频繁伸缩。

---