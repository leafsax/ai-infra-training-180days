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

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 推理服务）**：
  ```mermaid
  graph TD
      A[客户端/用户] --> B[API网关 / 负载均衡器]
      B --> C[vLLM/TGI 推理引擎]
      C --> D[GPU集群 H100/A100]
      C --> E[模型权重存储 NVMe/S3]
      C --> F[KV Cache 管理器]
      D --> G[GPU监控 DCGM]
  ```

- **数据流图（Data Flow Diagram - 推理流程）**：
  ```mermaid
  flowchart LR
      A[用户提示词] --> B[API网关]
      B --> C[请求队列与批处理]
      C --> D[GPU推理计算]
      D --> E[Token生成]
      E --> F[响应流式输出]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于推理批处理与延迟**：你提到了静态批处理与连续批处理。在中途动态拆分批处理时，内核融合（kernel fusion）的确切开销是多少？当长上下文请求（如128K token）到达并迫使为较短请求进行KV cache驱逐时，如何保证TP99延迟？
- **关于分布式训练通信**：在DP/TP/PP中，如何在1024个GPU的集群中处理straggler（慢节点）节点？如果一个GPU慢5%，整个All-Reduce步骤会阻塞吗？对于使用BF16权重和梯度的NCCL All-Reduce，每步的确切通信量是多少？
- **关于内存优化**：你提到了ZeRO和PagedAttention。对于ZeRO-3，如何在优化器步骤期间最小化跨节点通信开销？对于PagedAttention，当HBM发生碎片化时，确切的驱逐策略是什么（LRU vs LFU vs 基于注意力分数）？
