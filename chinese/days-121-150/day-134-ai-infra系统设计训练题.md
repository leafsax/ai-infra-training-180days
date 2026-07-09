### Day 134: 作业放置与亲和性/反亲和性调度
**1) 题目与考察核心**  
设计AI作业的Placement（放置）策略，利用Affinity/Anti-affinity优化网络与存储性能。

**2) 需求澄清与指标定义**  
- 作业：8x GPU分布式训练。
- 目标：同一作业的GPU尽量放在同一Node或同一Switch下，减少跨节点通信延迟。

**3) 核心架构/技术组件设计**  
- Node Affinity: 将Pod/Job调度到支持NVSwitch的节点。
- Topology-Aware Scheduling: 调度器感知网络拓扑（如NUMA, Switch Group）。

**4) 关键技术深入与可能解**  
- **NUMA Affinity**: CPU核与GPU在同一NUMA节点，减少跨CPU内存访问延迟。
- **Anti-Affinity**: 不同作业的Pod分散到不同节点，避免资源竞争。

**5) Trade-off（权衡）分析**  
- 强Affinity可能增加调度失败率（需特定拓扑资源）；Anti-Affinity提高资源利用率但可能增加通信延迟。

**6) 如何确定最优解**  
分布式训练作业启用Topology-Aware Scheduling，确保同一Job的GPU在同一Node或同一Leaf Switch下。

**7) 名词和缩写解释**  
- **Placement**: 作业放置，决定作业运行在哪些节点。
- **Affinity**: 亲和性，调度器倾向于将相关Pod放在同一节点。
- **Anti-Affinity**: 反亲和性，倾向于分散Pod。
- **NUMA**: Non-Uniform Memory Access（非统一内存访问）。

---

