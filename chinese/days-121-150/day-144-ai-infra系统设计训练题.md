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

