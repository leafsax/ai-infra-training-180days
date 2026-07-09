### Day 147: 容错与自动重启机制
**1) 题目与考察核心**  
设计AI训练的Fault Tolerance（容错）和Auto-Restart机制。

**2) 需求澄清与指标定义**  
- 故障率：万卡集群节点故障率约 1-2%/月。
- 目标：故障后自动恢复，损失Checkpoint < 1次保存周期。

**3) 核心架构/技术组件设计**  
- 作业监控：Slurm/K8s检测节点宕机，自动重启Job。
- 框架层：PyTorch `torch.distributed.run` 支持节点重启重加入。

**4) 关键技术深入与可能解**  
- 从最新Checkpoint恢复 vs 重算：Checkpoint恢复成本低。

**5) Trade-off（权衡）分析**  
- 频繁Checkpoint增加I/O开销；低频Checkpoint增加故障恢复损失。

**6) 如何确定最优解**  
配置每2小时或每1k steps自动Checkpoint，配合调度器自动重启。

**7) 名词和缩写解释**  
- **Fault Tolerance**: 容错能力。
- **Auto-Restart**: 自动重启机制。

---

