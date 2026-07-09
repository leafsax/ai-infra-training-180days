### Day 144: FSDP (Fully Sharded Data Parallel) Checkpointing
**1) 题目与考察核心**  
设计PyTorch FSDP的Checkpoint策略，优化保存和加载时间。

**2) 需求澄清与指标定义**  
- FSDP将模型参数分片到Data Parallel进程。
- Checkpoint需合并全量参数或保存分片。

**3) 核心架构/技术组件设计**  
- `save_sharded_state` vs `save_full_state`: 分片保存 vs 全量合并保存。
- `sharded_state_dict` API。

**4) 关键技术深入与可能解**  
- 分片保存节省IO和内存，但加载时需重新Shard。

**5) Trade-off（权衡）分析**  
- 全量保存兼容性好但IO大；分片保存高效但需FSDP环境恢复。

**6) 如何确定最优解**  
训练环境恢复时使用分片保存（`sharded_state_dict`），跨框架迁移时使用全量保存。

**7) 名词和缩写解释**  
- **FSDP**: Fully Sharded Data Parallel (PyTorch技术)。
- **DP**: Data Parallelism（数据并行）。

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
