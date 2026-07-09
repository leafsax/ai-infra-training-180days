### Day 145: Checkpoint 压缩与分层存储（Hot/Warm/Cold）
**1) 题目与考察核心**  
设计Checkpoint的压缩与Tiered Storage（分层存储）架构，降低成本。

**2) 需求澄清与指标定义**  
- 存储分层：Hot (NVMe), Warm (S3 Standard), Cold (S3 Glacier)。
- 压缩：Checkpoint压缩率目标 > 50%。

**3) 核心架构/技术组件设计**  
- 压缩算法：ZIP, GZIP, 或专用权重压缩（如FP8量化临时保存）。
- 自动化策略：最近Checkpoint存NVMe/S3 Standard，旧Checkpoint迁移到Glacier。

**4) 关键技术深入与可能解**  
- 量化压缩：保存FP16/BF16为INT8/FP8，恢复时转回。

**5) Trade-off（权衡）分析**  
- 压缩增加CPU开销和恢复时间；分层存储降低成本但Cold层恢复延迟高。

**6) 如何确定最优解**  
热Checkpoint存高速存储，历史Checkpoint压缩后归档至冷存储。

**7) 名词和缩写解释**  
- **Tiered Storage**: 分层存储（Hot/Warm/Cold）。
- **FP8/INT8**: 8位浮点/整数量化格式。

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
