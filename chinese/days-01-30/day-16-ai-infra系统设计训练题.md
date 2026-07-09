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


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于推理批处理与延迟**：你提到了静态批处理与连续批处理。在中途动态拆分批处理时，内核融合（kernel fusion）的确切开销是多少？当长上下文请求（如128K token）到达并迫使为较短请求进行KV cache驱逐时，如何保证TP99延迟？
- **关于分布式训练通信**：在DP/TP/PP中，如何在1024个GPU的集群中处理straggler（慢节点）节点？如果一个GPU慢5%，整个All-Reduce步骤会阻塞吗？对于使用BF16权重和梯度的NCCL All-Reduce，每步的确切通信量是多少？
- **关于内存优化**：你提到了ZeRO和PagedAttention。对于ZeRO-3，如何在优化器步骤期间最小化跨节点通信开销？对于PagedAttention，当HBM发生碎片化时，确切的驱逐策略是什么（LRU vs LFU vs 基于注意力分数）？
