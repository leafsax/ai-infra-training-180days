### Day 135: 弹性训练与动态资源扩展
**1) 题目与考察核心**  
设计Elastic Training（弹性训练）机制，支持训练过程中动态增减GPU资源。

**2) 需求澄清与指标定义**  
- 场景：集群资源紧张时，作业可动态缩容；资源释放时，可扩容。
- 目标：训练过程不丢失进度，通信库支持动态拓扑变化。

**3) 核心架构/技术组件设计**  
- 框架支持：PyTorch Elastic (`torch.distributed.elastic`), Ray Train。
- 通信库支持：NCCL支持动态节点加入/离开（需重启或动态重配置）。

**4) 关键技术深入与可能解**  
- **Reconfiguration**: 动态改变All-Reduce树结构。
- **Checkpoints + Resumption**: 缩容时保存Checkpoint，扩容后恢复。

**5) Trade-off（权衡）分析**  
- 弹性训练增加框架复杂度，且NCCL动态重配置可能需重启进程；静态分配更稳定但资源利用率低。

**6) 如何确定最优解**  
对于长周期训练，优先采用静态Gang Scheduling；对于批处理或实验作业，使用PyTorch Elastic + 自动Checkpoint。

**7) 名词和缩写解释**  
- **Elastic Training**: 弹性训练，训练过程中动态调整资源。
- **PyTorch Elastic**: PyTorch提供的弹性分布式训练库。

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
