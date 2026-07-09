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

### **8) 组件图与数据流图**

- **组件图（Component Diagram - GPU集群管理）**：
  ```mermaid
  graph TD
      A[集群控制器] --> B[GPU节点 H100]
      B --> C[MIG / vGPU 分区器]
      B --> D[RDMA 网络交换机]
      A --> E[自动扩缩容 KEDA]
      E --> B
  ```

- **数据流图（Data Flow Diagram - 扩缩容流程）**：
  ```mermaid
  flowchart LR
      A[流量突增] --> B[指标收集器 DCGM]
      B --> C[Scaler 控制器]
      C --> D[ Provision 新GPU Pods]
      D --> E[路由流量到新Pods]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于KV Cache与PagedAttention**：你提出了使用PagedAttention进行KV cache管理。当在不同上下文长度下HBM严重碎片化时会发生什么？在重负载下，你如何处理缓存驱逐策略，而不降低模型准确性或导致幻觉？
- **关于GPU分区（MIG/vGPU）**：对于MIG或vGPU分区，硬性隔离保证是什么？一个MIG分区中的噪音邻居是否仍会通过共享PCIe通道或CPU内存带宽影响另一个分区？如何实时测量和执行每个分区的SLO？
- **关于自动扩缩容与抖动（Thrashing）**：对于基于KEDA的自动扩缩容，启动新的vLLM pod的从零扩缩容延迟（scale-from-zero latency）是多少？当QPS在扩缩容阈值附近快速波动时（例如10秒内+20%然后-20%），如何防止扩缩容抖动？
