### Day 128: 对象存储 vs 块存储 vs 文件存储 for AI
**1) 题目与考察核心**  
区分对象存储、块存储、文件存储在AI训练、微调、推理场景中的应用。

**2) 需求澄清与指标定义**  
- 训练数据：TB级文件，顺序读取为主。
- 模型Checkpoint：GB级文件，需高可靠、低延迟写入。
- 虚拟机/容器镜像：GB级，随机读取。

**3) 核心架构/技术组件设计**  
- 文件存储（Lustre/NFS）：用于训练数据读取。
- 对象存储（S3/OSS）：用于长期归档、Checkpoint异步备份。
- 块存储（NVMe/SSD SAN）：用于虚拟机根盘、数据库。

**4) 关键技术深入与可能解**  
- **NFS vs 并行文件系统**：NFS简单但并发性能差；并行文件系统吞吐高但复杂。
- **S3异步上传**: 使用异步线程将Checkpoint写入S3，不阻塞训练主循环。

**5) Trade-off（权衡）分析**  
- 文件存储延迟低但扩展性差；对象存储扩展性强但随机读取延迟高；块存储性能高但无法多节点共享（除非SAN）。

**6) 如何确定最优解**  
训练数据用并行文件系统，Checkpoint用本地NVMe + 异步S3备份，容器镜像用块存储或本地Registry。

**7) 名词和缩写解释**  
- **S3**: Amazon Simple Storage Service。
- **NFS**: Network File System。
- **Checkpoint**: 训练过程中保存的模型状态（权重、优化器状态等）。
- **SAN**: Storage Area Network（存储区域网络）。

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
