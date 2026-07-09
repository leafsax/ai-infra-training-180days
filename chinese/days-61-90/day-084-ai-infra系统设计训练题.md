### Day 84: GPU 感知的自动扩缩容 (GPU-Aware Auto-Scaling)

**1) 题目与考察核心**：
设计 GPU 感知的调度与扩缩容，考察 GPU 利用率、HBM 使用与节点调度。

**2) 需求澄清与指标定义**：
- GPU 指标：GPU Utilization, HBM Usage, Temperature。
- 扩缩容目标：保持 GPU 利用率在 60-80%。

**3) 核心架构/技术组件设计**：
- GPU 监控：DCGM (Data Center GPU Manager) 收集指标。
- 调度器：K8s GPU 调度器（如 device-plugin）。

**4) 关键技术深入与可能解**：
- 基于 GPU Util vs HBM Usage：HBM 满会导致 OOM，即使 GPU Util 低。

**5) Trade-off（权衡）分析**：
- 激进扩缩容 vs 资源浪费。

**6) 如何确定最优解**：
监控 GPU Util + HBM Usage + 队列长度，设置 HBM 阈值触发扩容。

**7) 名词和缩写全称及解释**：
- **DCGM (Data Center GPU Manager)**：NVIDIA GPU 监控与管理工具。
- **HBM (High Bandwidth Memory)**：高带宽内存，GPU 使用的显存类型。

---

