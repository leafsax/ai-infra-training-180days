### Day 143: ZeRO-Offload 与 ZeRO-3 Checkpoint 存储优化
**1) 题目与考察核心**  
优化ZeRO-3和ZeRO-Offload的Checkpoint保存与加载性能。

**2) 需求澄清与指标定义**  
- ZeRO-3将模型、梯度、优化器状态分片到各GPU。
- Checkpoint合并需All-Gather，带宽压力大。

**3) 核心架构/技术组件设计**  
- Checkpoint合并：使用`state_dict()` 分片保存，或统一合并后保存。
- ZeRO-Offload：将优化器状态Offload到CPU内存/NVMe。

**4) 关键技术深入与可能解**  
- ZeRO-3 Checkpoint需各节点收集分片，使用分布式文件系统并行写入。

**5) Trade-off（权衡）分析**  
- 分片保存节省内存但增加元数据管理复杂度；统一保存简单但需大内存。

**6) 如何确定最优解**  
使用DeepSpeed ZeRO-3原生Checkpoint API，自动处理分片合并与分布式写入。

**7) 名词和缩写解释**  
- **ZeRO-Offload**: DeepSpeed技术，将优化器状态Offload到CPU。
- **ZeRO-3**: 分片模型、梯度、优化器状态。

---

