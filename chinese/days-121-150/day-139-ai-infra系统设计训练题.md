### Day 139: InfiniBand 拥塞控制与 QoS
**1) 题目与考察核心**  
设计InfiniBand网络的拥塞控制和QoS策略，优化All-Reduce通信。

**2) 需求澄清与指标定义**  
- IB网络：HDR or NDR InfiniBand (400G/800G)。
- QoS：为AI训练流量分配高优先级。

**3) 核心架构/技术组件设计**  
- IB Congestion Control：基于VC (Virtual Channel) 和 CC (Congestion Control) 机制。
- QoS：通过Priority字段标记AI训练流量。

**4) 关键技术深入与可能解**  
- IB内置拥塞控制，无需外部PFC配置，比RoCEv2更稳定。

**5) Trade-off（权衡）分析**  
- IB拥塞控制透明但封闭；RoCEv2开放但需调优。

**6) 如何确定最优解**  
追求稳定且预算充足时，首选InfiniBand + 内置CC机制。

**7) 名词和缩写解释**  
- **QoS**: Quality of Service。
- **VC**: Virtual Channel（虚拟通道）。
- **HDR/NDR**: InfiniBand速率标准（HDR=200G, NDR=400G, XDR=800G）。

---

