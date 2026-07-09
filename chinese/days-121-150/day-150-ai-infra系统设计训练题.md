### Day 150: AI 基础设施可观测性与 SLO
**1) 题目与考察核心**  
设计AI集群的可观测性（Observability）体系与SLO（服务等级目标）。

**2) 需求澄清与指标定义**  
- SLO目标：训练作业成功率 > 95%，平均故障恢复时间 < 30分钟。
- 监控维度：Compute (GPU Util), Network (Bw, Latency), Storage (I/O), Scheduling (Wait Time)。

**3) 核心架构/技术组件设计**  
- 监控栈：Prometheus + Grafana + 自定义AI Metrics Exporter。
- 日志与追踪：ELK/EFK + Jaeger。

**4) 关键技术深入与可能解**  
- 指标采集：node-exporter, dcmi (NVIDIA管理工具), NCCL stats。

**5) Trade-off（权衡）分析**  
- 高粒度监控增加存储和计算开销；低粒度可能漏掉瞬态故障。

**6) 如何确定最优解**  
采用Prometheus + 每秒/每分钟粒度指标，结合告警规则（如GPU Down, NCCL Timeout）。

**7) 名词和缩写解释**  
- **Observability**: 可观测性，通过指标、日志、追踪了解系统状态。
- **SLO**: Service Level Objective（服务等级目标）。
- **Prometheus**: 开源监控与告警工具。
- **Grafana**: 可视化监控面板工具。
- **dcmi**: NVIDIA Data Center Management Interface。

---

## 总结
- **完成内容**：生成了Days 121-150的AI Infra System Design Training Question Set，共计30天，主题涵盖网络架构、存储系统与任务调度。
- **每日结构**：每一天均包含7个部分：题目与考察核心、需求澄清与指标定义、核心架构/技术组件设计、关键技术深入与可能解、Trade-off分析、如何确定最优解、名词和缩写的全称及解释。
- **涵盖主题**：网络拓扑（Fat-Tree, Dragonfly+）、InfiniBand vs RoCEv2、RDMA/GDR、NVLink/NVSwitch、NCCL、分布式文件系统（Lustre/GPFS/Ceph）、NVMe缓存、数据Pipeline、Slurm vs K8s、Gang Scheduling、Preemption、Affinity、弹性训练、RDMA QP类型、UCX、拥塞控制（PFC/ECN/DCQCN）、Telemetry（P4/eBPF）、Checkpointing（同步/异步、ZeRO-3、FSDP）、分层存储、多租户隔离、容错与慢节点检测、GPU分区（MIG/vGPU）、可观测性与SLO。
- **文件格式**：全部内容已格式化为Markdown文本并按天列出。

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
