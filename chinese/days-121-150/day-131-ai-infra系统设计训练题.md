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

