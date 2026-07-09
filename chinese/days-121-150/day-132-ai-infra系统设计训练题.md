### Day 132: 分布式训练的 Gang Scheduling（批量调度）
**1) 题目与考察核心**  
设计Gang Scheduling机制，确保分布式训练作业的所有节点同时启动，避免死锁或资源碎片。

**2) 需求澄清与指标定义**  
- 作业规模：需分配256个GPU节点。
- 调度目标：100%成功调度或全部拒绝（All-or-Nothing）。

**3) 核心架构/技术组件设计**  
- 调度器扩展：在Slurm中使用`--mnodes`, `--nodes`；在K8s中使用Volcano Gang Scheduling Plugin。
- 资源预留：Reservation机制。

**4) 关键技术深入与可能解**  
- **All-or-Nothing**: 确保所有所需资源可用才启动，避免部分节点启动后等待。
- **Backfilling**: 在空闲资源中插入短作业，同时不干扰大作业。

**5) Trade-off（权衡）分析**  
- All-or-Nothing减少资源碎片但可能增加长作业等待时间；Backfilling提高利用率但调度逻辑复杂。

**6) 如何确定最优解**  
AI训练作业必须使用Gang Scheduling（All-or-Nothing），配合集群资源Reservation。

**7) 名词和缩写解释**  
- **Gang Scheduling**: 批量调度，确保作业所有部分同时运行。
- **All-or-Nothing**: 调度策略，全部资源满足才启动。
- **Backfilling**: 调度算法，利用空闲碎片资源运行短作业。

---

