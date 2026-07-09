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



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 任务调度）**：
  ```mermaid
  graph TD
      A[用户作业提交] --> B[调度器 K8s/Slurm]
      B --> C[Gang 调度控制器]
      C --> D[资源亲和性检查器]
      D --> E[GPU节点分配]
      E --> F[作业执行]
  ```

- **数据流图（Data Flow Diagram - 调度流程）**：
  ```mermaid
  flowchart LR
      A[作业请求] --> B[队列与优先级排序]
      B --> C[资源可用性检查]
      C --> D[分配 Gang Pods]
      D --> E[启动训练作业]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于网络可靠性与PFC风暴**：对于带有DCQCN的RoCEv2或InfiniBand，当发生拥塞时，如何防止PFC（优先级流控制）风暴？当800G NIC或交换链路意外flap时，确切的恢复时间和数据丢失场景是什么？
- **关于分布式存储I/O**：对于Lustre/GPFS/Ceph等分布式文件系统，如何处理多个GPU节点同时产生的小随机I/O模式？元数据服务器（metadata server）的瓶颈是什么，如何在不引入单点故障的情况下进行扩展？
- **关于Gang Scheduling与抢占**：对于K8s/Slurm中的Gang Scheduling，如果所需gang中的一个节点不健康或被驱逐，会发生什么？如何在不同租户队列之间处理抢占公平性，以防止长期运行的训练作业被饥饿？
