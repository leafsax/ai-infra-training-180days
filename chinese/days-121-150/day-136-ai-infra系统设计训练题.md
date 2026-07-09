### Day 136: RDMA 语义：RDMAP, RC, UC, UD, XRC
**1) 题目与考察核心**  
深入RDMA通信原语与QP (Queue Pair) 类型，为不同AI通信模式选择合适的RDMA类型。

**2) 需求澄清与指标定义**  
- 通信模式：Point-to-Point (P2P), Broadcast, All-Reduce。
- 要求：低延迟、高吞吐、可靠传输。

**3) 核心架构/技术组件设计**  
- **RC (Reliable Connection)**: 可靠连接，用于All-Reduce等需要可靠传输的集合通信。
- **UC (Unreliable Connection)**: 无连接可靠投递（无重传），适合延迟敏感但可容忍少量丢包（通过上层重传）的场景。
- **UD (Unreliable Datagram)**: 不可靠数据报，最低延迟，无连接。

**4) 关键技术深入与可能解**  
- RC提供ACK和重传，UC/UD无重传。AI训练中的NCCL主要使用RC或UC（配合NCCL自身的可靠性机制）。

**5) Trade-off（权衡）分析**  
- RC可靠但延迟高、有ACK开销；UC/UD延迟低但需上层处理丢包。

**6) 如何确定最优解**  
NCCL在InfiniBand/RoCEv2上默认使用RC保证可靠性，或通过UCX配置UC以优化延迟。

**7) 名词和缩写解释**  
- **QP**: Queue Pair（队列对，RDMA通信端点）。
- **RC**: Reliable Connection（可靠连接）。
- **UC**: Unreliable Connection（无连接可靠投递）。
- **UD**: Unreliable Datagram（不可靠数据报）。
- **XRC**: Extended Reliable Connection（扩展可靠连接，支持多QD）。

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
