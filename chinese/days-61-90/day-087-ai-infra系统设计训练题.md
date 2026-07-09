### Day 87: 无服务器推理与冷启动缓解 (Serverless Inference and Cold Start Mitigation)

**1) 题目与考察核心**：
设计无服务器 LLM 推理服务，考察模型预热（Warming）与快照（Snapshotting）。

**2) 需求澄清与指标定义**：
- 冷启动延迟：目标 < 5 秒。
- 模型大小：7B 参数，约 14 GB (FP16)。

**3) 核心架构/技术组件设计**：
- 模型预热：维护最小数量的预热 Pod。
- 快照：使用 containerd snapshot 快速恢复实例。

**4) 关键技术深入与可能解**：
- 预热策略：固定预热 vs 预测性预热（基于历史 QPS 模式）。

**5) Trade-off（权衡）分析**：
- 固定预热增加空闲成本；预测性预热复杂但节省成本。

**6) 如何确定最优解**：
采用预测性预热 + 模型共享内存（如 vLLM 的 prefix caching）。

**7) 名词和缩写全称及解释**：
- **Cold Start**：无服务器或容器首次启动时的延迟。
- **Prefix Caching**：缓存请求的公共前缀 token，加速相似请求生成。

---

