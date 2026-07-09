### Day 138: RoCEv2 拥塞控制：PFC, ECN, DCQCN
**1) 题目与考察核心**  
设计RoCEv2无损网络的拥塞控制机制，避免PFC storm和吞吐下降。

**2) 需求澄清与指标定义**  
- 网络：800G RoCEv2以太网。
- 目标：丢包率 < 10^-5，避免PFC死锁。

**3) 核心架构/技术组件设计**  
- **PFC (Priority Flow Control)**: 链路层流控，按优先级暂停发送。
- **ECN (Explicit Congestion Notification)**: 网络层拥塞标记，端点减速。
- **DCQCN**: 结合PFC和ECN的拥塞控制算法。

**4) 关键技术深入与可能解**  
- PFC Storm：一个端口触发PFC，导致上游端口也触发PFC，引发全网流控死锁。
- ECN+DCQCN：交换机通过ECN标记数据包，发送端降低发送速率，避免PFC。

**5) Trade-off（权衡）分析**  
- 纯PFC简单但易死锁；ECN+DCQCN复杂但稳定，需交换机和端点共同支持。

**6) 如何确定最优解**  
RoCEv2无损网络推荐启用ECN+DCQCN，合理配置PFC优先级（仅数据流启用PFC，控制流禁用）。

**7) 名词和缩写解释**  
- **DCQCN**: Data Center Quantized Congestion Notification。
- **PFC Storm**: PFC流控风暴。

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
