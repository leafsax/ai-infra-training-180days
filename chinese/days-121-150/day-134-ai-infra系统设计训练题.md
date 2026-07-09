### Day 134: 作业放置与亲和性/反亲和性调度
**1) 题目与考察核心**  
设计AI作业的Placement（放置）策略，利用Affinity/Anti-affinity优化网络与存储性能。

**2) 需求澄清与指标定义**  
- 作业：8x GPU分布式训练。
- 目标：同一作业的GPU尽量放在同一Node或同一Switch下，减少跨节点通信延迟。

**3) 核心架构/技术组件设计**  
- Node Affinity: 将Pod/Job调度到支持NVSwitch的节点。
- Topology-Aware Scheduling: 调度器感知网络拓扑（如NUMA, Switch Group）。

**4) 关键技术深入与可能解**  
- **NUMA Affinity**: CPU核与GPU在同一NUMA节点，减少跨CPU内存访问延迟。
- **Anti-Affinity**: 不同作业的Pod分散到不同节点，避免资源竞争。

**5) Trade-off（权衡）分析**  
- 强Affinity可能增加调度失败率（需特定拓扑资源）；Anti-Affinity提高资源利用率但可能增加通信延迟。

**6) 如何确定最优解**  
分布式训练作业启用Topology-Aware Scheduling，确保同一Job的GPU在同一Node或同一Leaf Switch下。

**7) 名词和缩写解释**  
- **Placement**: 作业放置，决定作业运行在哪些节点。
- **Affinity**: 亲和性，调度器倾向于将相关Pod放在同一节点。
- **Anti-Affinity**: 反亲和性，倾向于分散Pod。
- **NUMA**: Non-Uniform Memory Access（非统一内存访问）。

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
