### Day 131: AI集群任务调度基础：Kubernetes vs Slurm
**1) 题目与考察核心**  
对比Kubernetes和Slurm在AI集群任务调度中的适用性。

**2) 需求澄清与指标定义**  
- 集群规模：1,000 - 10,000 GPU节点。
- 工作负载：长周期训练作业（数天至数周），批量数据处理作业。
- 调度目标：高利用率，低作业等待时间。

**3) 核心架构/技术组件设计**  
- **Slurm**: HPC传统调度器，支持Gang Scheduling，适合长周期批量作业。
- **Kubernetes (K8s)**: 云原生调度器，生态丰富，适合微服务、推理服务、弹性训练。

**4) 关键技术深入与可能解**  
- Slurm的`sbatch`与K8s的`Job/CronJob`。
- K8s调度扩展：Volcano, KubeFlow Pipelines, Yunikorn。

**5) Trade-off（权衡）分析**  
- Slurm：调度AI训练作业成熟，但云原生生态弱；K8s：生态强，但原生调度器对Gang Scheduling和长周期作业支持需插件。

**6) 如何确定最优解**  
HPC/AI训练集群首选Slurm或K8s+Volcano；推理和服务化场景首选K8s。

**7) 名词和缩写解释**  
- **Slurm**: Simple Linux Utility for Resource Management。
- **Kubernetes (K8s)**: 开源容器编排平台。
- **Gang Scheduling**: 批量调度，作业所有部分同时调度或都不调度。
- **Volcano**: 云原生批量调度系统，扩展K8s。

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
