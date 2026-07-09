## 第16天：分布式训练框架（DDP, FSDP, DeepSpeed）

### 1) 题目与考察核心
**题目**：选择并配置分布式训练框架以训练70B参数LLM。
**考察核心**：对比PyTorch DDP, FSDP, DeepSpeed ZeRO。

### 2) 需求澄清与指标定义
- **训练数据量**：1万亿Tokens。
- **指标**：训练吞吐量 > 50k Tokens/s/GPU，显存效率 > 85%。

### 3) 核心架构/技术组件设计
- **Framework Selection**：选择PyTorch + FSDP 或 DeepSpeed。
- **Compute Network**：GPU间使用NVLink/NCCL，节点间使用InfiniBand。

### 4) 关键技术深入与可能解
- *DDP*：简单但显存占用高（完整模型副本）。
- *FSDP*：PyTorch原生，支持ZeRO-3分片和Activation Checkpointing。
- *DeepSpeed ZeRO*：功能丰富，支持ZeRO-1/2/3和Offload，但集成复杂度高。

### 5) Trade-off（权衡）分析
- *DDP vs FSDP/ZeRO*：DDP速度快但显存受限；FSDP/ZeRO显存效率高但通信和调度开销增加。

### 6) 如何确定最优解
- 对于70B+模型，默认选择PyTorch FSDP或DeepSpeed ZeRO-3，配合Activation Checkpointing以平衡显存与计算。

### 7) 名词和缩写全称及解释
- **DDP (Distributed Data Parallel)**：分布式数据并行，PyTorch基础并行策略。
- **FSDP (Fully Sharded Data Parallel)**：PyTorch的完全分片数据并行实现。
- **DeepSpeed**：微软开源的深度学习优化库，提供ZeRO、混合精度等优化。

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
