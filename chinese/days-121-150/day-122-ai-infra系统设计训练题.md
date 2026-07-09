### Day 122: InfiniBand vs RoCEv2 在AI集群中的选型
**1) 题目与考察核心**  
对比InfiniBand和RoCEv2，为大规模LLM训练集群设计网络底层传输方案。

**2) 需求澄清与指标定义**  
- 规模：4,096 GPU节点。
- 带宽需求：单节点网卡800 Gbps。
- 延迟要求：P2P（点对点）通信延迟 < 1.5 μs，All-Reduce组播延迟优化。
- 可靠性：网络丢包率 < 10^-5，需无损网络（Lossless Network）。

**3) 核心架构/技术组件设计**  
- InfiniBand架构：使用Switch和Host Channel Adapters (HCAs)，原生支持RDMA，内置拥塞控制。
- RoCEv2架构：基于标准以太网交换机，通过配置PFC（优先级流控）和ECN实现无损传输。

**4) 关键技术深入与可能解**  
- **InfiniBand**: 原生RDMA，硬件级拥塞控制，低延迟，但生态封闭、成本高。
- **RoCEv2**: 基于IP网络的RDMA，成本低、生态开放，但需精细调优PFC（避免PFC storm）和ECN。

**5) Trade-off（权衡）分析**  
- 成本 vs 性能：InfiniBand性能稳定但昂贵；RoCEv2成本低但调优复杂，易出现PFC死锁或拥塞崩溃。
- 运维复杂度：InfiniBand运维简单但供应商锁定；RoCEv2可复用现有以太网运维体系，但需专门的RDMA调优团队。

**6) 如何确定最优解**  
若集群规模<2,000 GPU且追求稳定交付，选InfiniBand；若规模>4,000 GPU且需控制CapEx（资本支出），选RoCEv2并投入工程团队调优PFC/ECN/DCQCN。

**7) 名词和缩写解释**  
- **InfiniBand (IB)**: 高性能并行计算网络标准。
- **RoCEv2**: RDMA over Converged Ethernet version 2。
- **RDMA**: Remote Direct Memory Access（远程直接内存访问）。
- **HCA**: Host Channel Adapter（主机通道适配器，IB网络中的网卡）。
- **PFC**: Priority-based Flow Control（基于优先级的流控）。
- **PFC storm**: PFC流控风暴，因局部拥塞导致全网流控死锁。
- **ECN**: Explicit Congestion Notification（显式拥塞通知）。
- **CapEx**: Capital Expenditure（资本支出）。

---

