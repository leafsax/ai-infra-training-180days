## 第27天：自定义内核与CUDA优化基础

### 1) 题目与考察核心
**题目**：为特定LLM操作编写自定义CUDA/Triton内核以优化性能。
**考察核心**：CUDA编程基础，Triton语言，内存访问模式优化。

### 2) 需求澄清与指标定义
- **优化目标**：自定义Layer（如特殊Normalization）速度提升2x。
- **指标**：内核执行时间 < 100µs，内存带宽利用率 > 80%。

### 3) 核心架构/技术组件设计
- **Triton Kernel Development**：使用Triton DSL编写内核，编译为SASS/PTX部署到GPU。
- **Memory Coalescing**：优化全局内存访问模式以实现合并访问。

### 4) 关键技术深入与可能解
- *CUDA C++ vs Triton*：CUDA C++灵活但编写复杂；Triton抽象层次高，易于编写高效的Attention或Normalization内核。
- *Shared Memory Usage*：充分利用GPU的Shared Memory（SRAM）减少全局内存访问。

### 5) Trade-off（权衡）分析
- *通用性 vs 性能*：自定义内核性能极致但难以维护且需针对特定硬件（如H100 vs A100）调整；使用通用库（如CUTLASS）可移植性好但非绝对最优。

### 6) 如何确定最优解
- 优先使用成熟库（FlashAttention, Triton built-ins）；仅在现有库无法满足特定操作（如自定义激活函数）时编写Triton内核。

### 7) 名词和缩写全称及解释
- **CUDA (Compute Unified Device Architecture)**：NVIDIA的并行计算平台和编程模型。
- **Triton DSL**：Triton领域特定语言，用于编写GPU内核。
- **Memory Coalescing**：内存合并访问，GPU线程以连续方式访问全局内存以提高带宽利用率。

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
