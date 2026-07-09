### Day 142: 同步 vs 异步 Checkpointing
**1) 题目与考察核心**  
对比同步和异步Checkpoint机制，设计高可用训练流水线。

**2) 需求澄清与指标定义**  
- 目标：Checkpoint操作不占用训练Compute时间。

**3) 核心架构/技术组件设计**  
- 异步Checkpoint线程/进程：独立于训练主循环。
- 存储：本地NVMe -> 远端分布式存储。

**4) 关键技术深入与可能解**  
- 同步：`torch.save()` 阻塞；异步：使用后台线程或`torch.distributed.checkpoint.async_save`。

**5) Trade-off（权衡）分析**  
- 异步增加代码复杂度和状态同步难度。

**6) 如何确定最优解**  
使用框架原生异步Checkpoint API（如FSDP async save）。

**7) 名词和缩写解释**  
- **Synchronous Checkpointing**: 同步保存，阻塞训练。
- **Asynchronous Checkpointing**: 异步保存，后台进行。

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
