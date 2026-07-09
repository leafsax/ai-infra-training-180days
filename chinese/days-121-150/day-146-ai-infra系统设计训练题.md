### Day 146: AI集群多租户与资源隔离
**1) 题目与考察核心**  
设计多租户AI集群的资源隔离与QoS机制。

**2) 需求澄清与指标定义**  
- 租户：多个团队/项目共享万卡集群。
- 隔离目标：网络、存储、计算资源隔离，防止干扰。

**3) 核心架构/技术组件设计**  
- 计算隔离：K8s Namespace + Resource Quota。
- 网络隔离：VLAN, VRF, 或IB Partition。

**4) 关键技术深入与可能解**  
- IB Partition：InfiniBand硬件级网络隔离。
- K8s Network Policies：软件定义网络隔离。

**5) Trade-off（权衡）分析**  
- 硬件隔离（IB Partition）安全但灵活度低；软件隔离灵活但可能有性能开销。

**6) 如何确定最优解**  
关键生产租户使用IB Partition或独立Leaf Switch群；实验租户使用K8s Namespace+QoS。

**7) 名词和缩写解释**  
- **VLAN**: Virtual Local Area Network。
- **VRF**: Virtual Routing and Forwarding。
- **IB Partition**: InfiniBand Partition（物理/逻辑网络隔离）。

---

