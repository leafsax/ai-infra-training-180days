### Day 133: 抢占式调度与AI集群公平调度
**1) 题目与考察核心**  
设计支持Preemption（抢占）和Fair Scheduling（公平调度）的AI集群调度策略。

**2) 需求澄清与指标定义**  
- 多租户环境：多个团队共享10,000 GPU集群。
- 优先级：Production训练作业 vs Experiment作业。
- 目标：高优先级作业可抢占低优先级作业的GPU资源。

**3) 核心架构/技术组件设计**  
- Slurm QoS (Quality of Service) 与 Preemption。
- K8s PriorityClass + Preemptive Scheduler。

**4) 关键技术深入与可能解**  
- **Preemption**: 高优先级作业到来时，终止或挂起低优先级作业释放资源。
- **Fair Sharing**: 按团队配额分配资源，防止单一团队垄断。

**5) Trade-off（权衡）分析**  
- Preemption增加作业中断风险，需配合Checkpoint自动恢复；Fair Sharing可能降低高优先级作业的绝对资源保障。

**6) 如何确定最优解**  
使用QoS分级，Production作业高优先级+自动Checkpoint恢复；按团队配额实施Fair Sharing。

**7) 名词和缩写解释**  
- **Preemption**: 资源抢占，高优先级任务中断低优先级任务。
- **Fair Scheduling**: 公平调度，按配额或权重分配资源。
- **QoS**: Quality of Service（服务质量）。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 网络架构）**：
  ```mermaid
  graph TD
      A[作业调度器 Slurm/K8s] --> B[计算节点 GPU+CPU]
      B --> C[RDMA网络 InfiniBand/RoCE]
      B --> D[分布式存储 Lustre/GPFS]
      C --> E[All-Reduce通信 NCCL]
  ```

- **数据流图（Data Flow Diagram - 训练数据与计算）**：
  ```mermaid
  flowchart LR
      A[数据加载器] --> B[本地 NVMe 缓存]
      B --> C[GPU计算内核]
      C --> D[NCCL All-Reduce via RDMA]
      D --> C
      C --> E[梯度更新]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于网络可靠性与PFC风暴**：对于带有DCQCN的RoCEv2或InfiniBand，当发生拥塞时，如何防止PFC（优先级流控制）风暴？当800G NIC或交换链路意外flap时，确切的恢复时间和数据丢失场景是什么？
- **关于分布式存储I/O**：对于Lustre/GPFS/Ceph等分布式文件系统，如何处理多个GPU节点同时产生的小随机I/O模式？元数据服务器（metadata server）的瓶颈是什么，如何在不引入单点故障的情况下进行扩展？
- **关于Gang Scheduling与抢占**：对于K8s/Slurm中的Gang Scheduling，如果所需gang中的一个节点不健康或被驱逐，会发生什么？如何在不同租户队列之间处理抢占公平性，以防止长期运行的训练作业被饥饿？
