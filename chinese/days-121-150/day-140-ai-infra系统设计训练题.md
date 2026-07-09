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
