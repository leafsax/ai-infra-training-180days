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