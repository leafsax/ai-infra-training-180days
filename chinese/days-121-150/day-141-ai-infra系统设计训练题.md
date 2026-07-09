### Day 141: 大规模训练中的 Checkpointing 挑战
**1) 题目与考察核心**  
设计万卡训练集群的Checkpoint机制，解决保存和恢复时的I/O与通信瓶颈。

**2) 需求澄清与指标定义**  
- Checkpoint频率：每1,000 steps或每2小时。
- Checkpoint大小：模型+优化器状态约 2 TB。
- 目标：Checkpoint保存时间 < 10分钟，不显著影响训练吞吐。

**3) 核心架构/技术组件设计**  
- 分布式Checkpoint：每个GPU/节点保存分片，并行写入存储。
- 框架：FSDP, ZeRO-3, Megatron-LM Checkpointing。

**4) 关键技术深入与可能解**  
- **Synchronous vs Asynchronous**: 同步Checkpoint阻塞训练；异步Checkpoint后台保存。

**5) Trade-off（权衡）分析**  
- 同步：数据一致性好但阻塞训练；异步：训练不阻塞但需处理版本一致性。

**6) 如何确定最优解**  
采用异步Checkpoint + 本地NVMe暂存 + 异步同步到分布式文件系统/S3。

**7) 名词和缩写解释**  
- **Checkpoint**: 模型状态保存文件。
- **FSDP**: Fully Sharded Data Parallel。
- **ZeRO-3**: Zero Redundancy Optimizer - Stage 3。

---

