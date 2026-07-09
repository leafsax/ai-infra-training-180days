### Day 136: RDMA 语义：RDMAP, RC, UC, UD, XRC
**1) 题目与考察核心**  
深入RDMA通信原语与QP (Queue Pair) 类型，为不同AI通信模式选择合适的RDMA类型。

**2) 需求澄清与指标定义**  
- 通信模式：Point-to-Point (P2P), Broadcast, All-Reduce。
- 要求：低延迟、高吞吐、可靠传输。

**3) 核心架构/技术组件设计**  
- **RC (Reliable Connection)**: 可靠连接，用于All-Reduce等需要可靠传输的集合通信。
- **UC (Unreliable Connection)**: 无连接可靠投递（无重传），适合延迟敏感但可容忍少量丢包（通过上层重传）的场景。
- **UD (Unreliable Datagram)**: 不可靠数据报，最低延迟，无连接。

**4) 关键技术深入与可能解**  
- RC提供ACK和重传，UC/UD无重传。AI训练中的NCCL主要使用RC或UC（配合NCCL自身的可靠性机制）。

**5) Trade-off（权衡）分析**  
- RC可靠但延迟高、有ACK开销；UC/UD延迟低但需上层处理丢包。

**6) 如何确定最优解**  
NCCL在InfiniBand/RoCEv2上默认使用RC保证可靠性，或通过UCX配置UC以优化延迟。

**7) 名词和缩写解释**  
- **QP**: Queue Pair（队列对，RDMA通信端点）。
- **RC**: Reliable Connection（可靠连接）。
- **UC**: Unreliable Connection（无连接可靠投递）。
- **UD**: Unreliable Datagram（不可靠数据报）。
- **XRC**: Extended Reliable Connection（扩展可靠连接，支持多QD）。

---

