### Day 142: 同步 vs 异步 Checkpointing
**1) 题目与考察核心**  
对比同步和异步Checkpoint机制，设计高可用训练流水线。

**2) 需求澄清与指标定义**  
- 目标：Checkpoint操作不占用训练Compute时间。

**3) 核心架构/技术组件设计**  
- 异步Checkpoint线程/进程：独立于训练主循环。
- 存储：本地NVMe -> 远端分布式存储。

**4) 关键技术深入与可能解**  
- 同步：`torch.save()` 阻塞；异步：使用后台线程或`torch.distributed.checkpoint.async_save`。

**5) Trade-off（权衡）分析**  
- 异步增加代码复杂度和状态同步难度。

**6) 如何确定最优解**  
使用框架原生异步Checkpoint API（如FSDP async save）。

**7) 名词和缩写解释**  
- **Synchronous Checkpointing**: 同步保存，阻塞训练。
- **Asynchronous Checkpointing**: 异步保存，后台进行。

---

