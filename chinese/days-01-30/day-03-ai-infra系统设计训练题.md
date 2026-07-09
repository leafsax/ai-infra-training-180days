## 第3天：模型并行基础（DP, TP, PP, MP）

### 1) 题目与考察核心
**题目**：设计一个支持千亿参数（100B+）LLM的分布式推理/训练系统架构。
**考察核心**：理解模型并行（Model Parallelism, MP）的三大基本范式：数据并行（DP）、张量并行（TP）、流水线并行（PP）。

### 2) 需求澄清与指标定义
- **模型大小**：100B参数，FP16精度下权重约200GB。
- **单卡显存**：80GB HBM，单卡无法容纳完整模型。
- **指标**：训练/推理吞吐最大化，通信开销最小化。

### 3) 核心架构/技术组件设计
- **混合并行策略**：DP + TP + PP 三维并行。例如：8路TP（Tensor Parallelism），8路PP（Pipeline Parallelism），4路DP（Data Parallelism），共256张GPU。

### 4) 关键技术深入与可能解
- *Data Parallelism (DP)*：复制完整模型到多个GPU，输入数据分片。
- *Tensor Parallelism (TP)*：将单层网络的权重矩阵切分到多个GPU（如行切分、列切分）。
- *Pipeline Parallelism (PP)*：将模型的不同层分配给不同的GPU，形成流水线。

### 5) Trade-off（权衡）分析
- *DP vs TP*：DP通信简单（All-Reduce梯度），但需完整模型副本；TP通信频繁（GEMM间的All-Reduce或All-Gather），但单卡显存占用低。
- *PP的Bubble时间*：流水线存在空泡（Bubble），即部分GPU在等待上游梯度或激活时处于空闲。

### 6) 如何确定最优解
- 根据模型大小和GPU数量，优先使用TP切分单层（减少跨节点通信），然后使用PP切分层，最后使用DP提升吞吐。使用Auto-Parallel工具（如PyTorch FSDP, Megatron-LM）自动寻找最优切分。

### 7) 名词和缩写全称及解释
- **MP (Model Parallelism)**：模型并行，将模型切分到多个设备。
- **DP (Data Parallelism)**：数据并行，多个GPU持有模型完整副本，处理不同数据。
- **TP (Tensor Parallelism)**：张量并行，将张量（矩阵）切分到多个GPU。
- **PP (Pipeline Parallelism)**：流水线并行，将网络层切分到多个GPU。

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
