## 第19天：训练数据管道与加载优化

### 1) 题目与考察核心
**题目**：设计高吞吐的LLM训练数据加载管道，避免GPU因等待数据而空闲。
**考察核心**：DataLoader优化，Prefetching，Ray Data，数据预取与解码。

### 2) 需求澄清与指标定义
- **数据吞吐量**：需支持 50GB/s 的数据读取和解码速度。
- **指标**：GPU利用率 > 90%，Data Loading时间 < 10% per step。

### 3) 核心架构/技术组件设计
- **Multi-process Data Loading**：使用多进程（Multi-processing）DataLoader。
- **Prefetch Buffer**：预取缓冲区，提前加载下一个Batch的数据。

### 4) 关键技术深入与可能解
- *PyTorch DataLoader*：基础加载器，但可能成为瓶颈。
- *Ray Data / WebDataset*：分布式数据加载框架，支持从S3高效流式读取。

### 5) Trade-off（权衡）分析
- *内存 vs 速度*：增大Prefetch Buffer和Num_workers提高速度但增加CPU RAM占用。

### 6) 如何确定最优解
- 使用WebDataset或Ray Data处理海量JSONL/Parquet数据，结合PyTorch的`num_workers=4`和`pin_memory=True`。

### 7) 名词和缩写全称及解释
- **DataLoader**：PyTorch中用于加载数据的组件。
- **Prefetching (预取)**：提前将数据加载到内存或设备，以减少等待时间。
- **WebDataset**：用于大规模训练的高效数据格式和加载库。

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
