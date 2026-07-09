## 第20天：混合精度训练与梯度检查点

### 1) 题目与考察核心
**题目**：优化LLM训练显存与速度，采用混合精度和激活检查点技术。
**考察核心**：FP16/BF16混合精度，Gradient Checkpointing（激活检查点）。

### 2) 需求澄清与指标定义
- **精度格式**：BF16（Bfloat16）或 FP16。
- **指标**：显存减少50%，训练速度提升1.5-2倍。

### 3) 核心架构/技术组件设计
- **Automatic Mixed Precision (AMP)**：PyTorch的`torch.cuda.amp`，自动选择操作符的精度。
- **Activation Checkpointing**：不保存所有中间激活值，而是在Backward时重新计算（Recompute）。

### 4) 关键技术深入与可能解
- *BF16 vs FP16*：BF16范围与FP32相同，避免溢出；FP16精度更高但易溢出。LLM训练首选BF16。
- *Gradient Checkpointing*：只保存部分层的激活，Backward时重新计算其余层，显存节省可达50%，计算增加约33%。

### 5) Trade-off（权衡）分析
- *Checkpting vs 显存*：启用Activation Checkpointing大幅减少显存，但增加Forward/Backward的计算时间（Trade-off：显存换计算）。

### 6) 如何确定最优解
- 默认启用BF16 AMP和Activation Checkpointing（`gradient_checkpointing=True`），在显存允许的情况下减少Checkpting粒度以提速。

### 7) 名词和缩写全称及解释
- **FP16 (Float16)**：16位浮点数格式。
- **BF16 (Bfloat16)**：Google提出的16位浮点格式，保持与FP32相同的指数范围。
- **AMP (Automatic Mixed Precision)**：自动混合精度训练。
- **Gradient/Activation Checkpointing**：梯度/激活检查点，通过重新计算激活值来节省显存。

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
