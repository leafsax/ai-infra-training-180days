### Day 133: 抢占式调度与AI集群公平调度
**1) 题目与考察核心**  
设计支持Preemption（抢占）和Fair Scheduling（公平调度）的AI集群调度策略。

**2) 需求澄清与指标定义**  
- 多租户环境：多个团队共享10,000 GPU集群。
- 优先级：Production训练作业 vs Experiment作业。
- 目标：高优先级作业可抢占低优先级作业的GPU资源。

**3) 核心架构/技术组件设计**  
- Slurm QoS (Quality of Service) 与 Preemption。
- K8s PriorityClass + Preemptive Scheduler。

**4) 关键技术深入与可能解**  
- **Preemption**: 高优先级作业到来时，终止或挂起低优先级作业释放资源。
- **Fair Sharing**: 按团队配额分配资源，防止单一团队垄断。

**5) Trade-off（权衡）分析**  
- Preemption增加作业中断风险，需配合Checkpoint自动恢复；Fair Sharing可能降低高优先级作业的绝对资源保障。

**6) 如何确定最优解**  
使用QoS分级，Production作业高优先级+自动Checkpoint恢复；按团队配额实施Fair Sharing。

**7) 名词和缩写解释**  
- **Preemption**: 资源抢占，高优先级任务中断低优先级任务。
- **Fair Scheduling**: 公平调度，按配额或权重分配资源。
- **QoS**: Quality of Service（服务质量）。

---

