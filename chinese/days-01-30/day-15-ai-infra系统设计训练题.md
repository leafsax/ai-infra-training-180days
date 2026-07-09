## 第15天：高可用与容错（HA & Fault Tolerance）

### 1) 题目与考察核心
**题目**：设计高可用的LLM训练与推理集群，处理GPU故障、节点宕机。
**考察核心**：Checkpointing, Failover, 健康检查与自动恢复。

### 2) 需求澄清与指标定义
- **可用性目标**：99.99%。
- **故障恢复时间（RTO）**：训练Checkpoint恢复 < 10分钟；推理服务Failover < 30秒。

### 3) 核心架构/技术组件设计
- **Distributed Checkpointing**：定期保存模型状态到高速存储（如NFS, S3, Ceph）。
- **Health Monitor & Auto-restart**：监控GPU温度、NCCL状态，故障时重启Pod或迁移工作负载。

### 4) 关键技术深入与可能解
- *Training Checkpointing*：全量Checkpoint vs Incremental Checkpoint。
- *Inference Failover*：Router检测到Engine无响应，立即将请求路由到备用Engine。

### 5) Trade-off（权衡）分析
- *Checkpoint频率*：高频Checkpoint保证恢复点近（RPO小）但增加I/O开销和训练停顿时间；低频Checkpoint相反。

### 6) 如何确定最优解
- 训练采用分布式Checkpoint（如DeepSpeed FSDP Checkpoint），每1000步保存一次，并异步写入对象存储。推理采用多副本（Replica）+ Router健康检查。

### 7) 名词和缩写全称及解释
- **HA (High Availability)**：高可用性，系统持续提供服务的能力。
- **RTO (Recovery Time Objective)**：恢复时间目标，从故障到恢复服务的时间。
- **RPO (Recovery Point Objective)**：恢复点目标，允许丢失的数据量（时间维度）。

---