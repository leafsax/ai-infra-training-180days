### Day 46: GPU集群成本优化：Spot Instances, 资源Right-sizing

**1) 题目与考察核心**
设计低成本GPU集群运营方案。考察核心：Spot实例使用、资源Right-sizing与闲置资源回收。

**2) 需求澄清与指标定义**
- **成本目标**：降低GPU采购与运行成本30%。
- **任务类型**：区分在线推理（Online）与离线批处理（Batch Inference）。

**3) 核心架构/技术组件设计**
- 任务调度层：在线推理部署在On-Demand实例，批处理推理部署在Spot Instances。
- 资源监控：识别低利用率Pod并自动缩容或终止。

**4) 关键技术深入与可能解**
- **Spot Instances**：云厂商的闲置GPU竞价实例，成本低但可能被回收。
- **Right-sizing**：根据实际GPU利用率调整Pod的请求显存与卡数，避免过度分配。

**5) Trade-off（权衡）分析**
- Spot实例成本低但存在中断风险；Right-sizing提高利用率但需持续监控与调优。

**6) 如何确定最优解**
在线推理服务使用On-Demand或预留实例；非实时批处理推理与训练任务使用Spot Instances，并实现自动迁移与重试机制。

**7) 名词和缩写全称及解释**
- **Spot Instances**：竞价实例，利用云厂商闲置计算资源，价格低但可能被提前回收。
- **On-Demand Instances**：按小时计费的常规云实例。

---