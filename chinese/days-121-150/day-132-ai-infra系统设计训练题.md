### Day 132: 分布式训练的 Gang Scheduling（批量调度）
**1) 题目与考察核心**  
设计Gang Scheduling机制，确保分布式训练作业的所有节点同时启动，避免死锁或资源碎片。

**2) 需求澄清与指标定义**  
- 作业规模：需分配256个GPU节点。
- 调度目标：100%成功调度或全部拒绝（All-or-Nothing）。

**3) 核心架构/技术组件设计**  
- 调度器扩展：在Slurm中使用`--mnodes`, `--nodes`；在K8s中使用Volcano Gang Scheduling Plugin。
- 资源预留：Reservation机制。

**4) 关键技术深入与可能解**  
- **All-or-Nothing**: 确保所有所需资源可用才启动，避免部分节点启动后等待。
- **Backfilling**: 在空闲资源中插入短作业，同时不干扰大作业。

**5) Trade-off（权衡）分析**  
- All-or-Nothing减少资源碎片但可能增加长作业等待时间；Backfilling提高利用率但调度逻辑复杂。

**6) 如何确定最优解**  
AI训练作业必须使用Gang Scheduling（All-or-Nothing），配合集群资源Reservation。

**7) 名词和缩写解释**  
- **Gang Scheduling**: 批量调度，确保作业所有部分同时运行。
- **All-or-Nothing**: 调度策略，全部资源满足才启动。
- **Backfilling**: 调度算法，利用空闲碎片资源运行短作业。

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
