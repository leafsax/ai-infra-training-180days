## 第25天：AI Infra成本优化（Spot Instances, 资源右化）

### 1) 题目与考察核心
**题目**：设计成本优化的AI训练与推理集群架构。
**考察核心**：Spot Instances（竞价实例），GPU Right-sizing，空闲资源回收。

### 2) 需求澄清与指标定义
- **成本目标**：训练成本降低40%。
- **指标**：Spot实例中断率 < 5%/天，训练任务通过Checkpoint恢复成功率 > 99%。

### 3) 核心架构/技术组件设计
- **Spot + On-Demand Hybrid Cluster**：训练任务优先调度到Spot节点，配置高频率Checkpoint；推理任务使用On-Demand或Reserved实例。
- **GPU Sharing / Time-slicing**：在推理低峰期，通过MIG（Multi-Instance GPU）或时间片共享GPU。

### 4) 关键技术深入与可能解
- *Spot Instances*：云厂商的闲置GPU资源，价格低廉但可能被随时回收。
- *MIG (Multi-Instance GPU)*：将单张GPU物理切分为多个独立实例，适合多租户推理隔离。

### 5) Trade-off（权衡）分析
- *Spot vs On-Demand*：Spot成本低但不可靠；On-Demand可靠但昂贵。训练可容忍中断（通过Checkpoint），推理需高可用。

### 6) 如何确定最优解
- 训练集群主要使用Spot Instances + 自动恢复机制；推理集群使用On-Demand或长期预留实例（Reserved Instances）。

### 7) 名词和缩写全称及解释
- **Spot Instances (竞价实例)**：云服务商提供的打折闲置计算资源，可随时被回收。
- **On-Demand Instances**：按小时或秒计费的常规云实例，稳定可靠。
- **MIG (Multi-Instance GPU)**：NVIDIA GPU的多实例分割技术，将单卡切分为多个独立GPU实例。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 分布式训练）**：
  ```mermaid
  graph TD
      A[数据加载器] --> B[训练节点 GPU+CPU]
      B --> C[NCCL All-Reduce 网络]
      B --> D[检查点存储 S3/NFS]
      B --> E[模型注册表 MLflow]
      C --> B
  ```

- **数据流图（Data Flow Diagram - 训练流程）**：
  ```mermaid
  flowchart LR
      A[训练数据] --> B[各GPU梯度计算]
      B --> C[NCCL梯度同步]
      C --> D[权重更新]
      D --> E[检查点保存]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于推理批处理与延迟**：你提到了静态批处理与连续批处理。在中途动态拆分批处理时，内核融合（kernel fusion）的确切开销是多少？当长上下文请求（如128K token）到达并迫使为较短请求进行KV cache驱逐时，如何保证TP99延迟？
- **关于分布式训练通信**：在DP/TP/PP中，如何在1024个GPU的集群中处理straggler（慢节点）节点？如果一个GPU慢5%，整个All-Reduce步骤会阻塞吗？对于使用BF16权重和梯度的NCCL All-Reduce，每步的确切通信量是多少？
- **关于内存优化**：你提到了ZeRO和PagedAttention。对于ZeRO-3，如何在优化器步骤期间最小化跨节点通信开销？对于PagedAttention，当HBM发生碎片化时，确切的驱逐策略是什么（LRU vs LFU vs 基于注意力分数）？
