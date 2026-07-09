### Day 137: UCX (Unified Communication X) 与 AI 工作负载
**1) 题目与考察核心**  
设计基于UCX的通用通信栈，支持RDMA、TCP、SHM等多种传输协议。

**2) 需求澄清与指标定义**  
- 目标：统一GPU间（SHM）、节点内（NVLink）、节点间（RDMA/TCP）的通信接口。
- 性能：UCX底层传输自动选择最优路径。

**3) 核心架构/技术组件设计**  
- UCX组件：`ucp` (UCX Process), `uct` (UCX Transport), `ucm` (UCX Memory)。
- 传输优先级：SHM > NVLink > RDMA > TCP。

**4) 关键技术深入与可能解**  
- UCX的Transport Selection：根据拓扑自动选择SHM（进程间）、ib_rc（InfiniBand可靠连接）、rc_uc（RoCEv2）等。

**5) Trade-off（权衡）分析**  
- UCX提供灵活性但增加栈深度；原生NCCL/ MPI更专一但跨平台兼容性弱。

**6) 如何确定最优解**  
AI集群中，NCCL底层已集成UCX或自研RDMA栈；自定义通信应用推荐使用UCX。

**7) 名词和缩写解释**  
- **UCX**: Unified Communication X。
- **SHM**: Shared Memory（共享内存）。
- **ib_rc**: InfiniBand Reliable Connection。

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
