### Day 140: AI集群网络监控与遥测（P4, eBPF）
**1) 题目与考察核心**  
设计AI集群网络监控与Telemetry（遥测）系统，实时检测网络拥塞和慢节点。

**2) 需求澄清与指标定义**  
- 监控指标：网卡吞吐、丢包率、PFC次数、ECN标记数、RTT。
- 粒度：每秒/每端口指标。

**3) 核心架构/技术组件设计**  
- **P4可编程交换机**: 在交换机数据平面提取流量特征。
- **eBPF**: 在主机内核层监控网络栈和RDMA性能。

**4) 关键技术深入与可能解**  
- P4 vs eBPF：P4在交换机侧，eBPF在主机侧。结合使用实现端到端监控。

**5) Trade-off（权衡）分析**  
- P4需交换机支持且编程复杂；eBPF需内核支持且可能影响性能。

**6) 如何确定最优解**  
使用交换机Vendor提供的Telemetry API（如NVIDIA Spectrum-X）结合主机eBPF工具（如`ibstatus`, `rdma-stat`）。

**7) 名词和缩写解释**  
- **Telemetry**: 遥测，远程测量和发送数据。
- **P4**: Programming Protocol-independent Packet Processors。
- **eBPF**: extended Berkeley Packet Filter。

---

