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