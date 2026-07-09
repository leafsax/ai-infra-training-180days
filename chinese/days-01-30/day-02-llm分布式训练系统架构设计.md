# 第2天：LLM分布式训练系统架构设计

## 1) 题目与考察核心
**题目**：设计一个用于训练 100B 参数大语言模型的分布式训练系统。
**考察核心**：分布式训练并行策略（DP/TP/PP）、显存优化技术（ZeRO）、通信优化。

## 2) 需求澄清与指标定义
- **GPU 数量**：1024 张 H100 80GB GPU。
- **训练时间**：< 30 天完成预训练。
- **计算效率**：**TFLOPs Utilization（每秒万亿次浮点运算利用率）** > 60%。
- **模型参数**：100B（1000亿）参数，FP16/BF16 精度。

## 3) 核心架构/技术组件设计
- **数据并行（DP）节点集群**：将数据集分片分配到多个 GPU 上。
- **张量并行（TP）层**：将模型权重（如 Attention 矩阵、MLP 矩阵）切分到多个 GPU。
- **流水线并行（PP）阶段**：将模型的不同层分配给不同的 GPU 组，形成流水线。
- **优化器状态管理**：使用 ZeRO 技术减少优化器、梯度和参数的冗余存储。

## 4) 关键技术深入与可能解
- **DP（Data Parallel，数据并行）**：每个 GPU 拥有完整的模型副本，处理不同的数据分片，最后同步梯度。
- **TP（Tensor Parallel，张量并行）**：将单个大矩阵的乘法运算切分，多个 GPU 协同完成一次前向/反向传播。
- **PP（Pipeline Parallel，流水线并行）**：将模型层按顺序切分，不同 GPU 同时处理不同 Batch 的数据层。
- **ZeRO（Zero Redundancy Optimizer，零冗余优化器）**：由 DeepSpeed 提出，分为 ZeRO-1（优化器状态分片）、ZeRO-2（梯度分片）、ZeRO-3（参数、梯度、优化器状态全分片）。

## 5) Trade-off（权衡）分析
- **DP vs TP vs PP**：DP 通信量小但显存占用高；TP 显存占用低但 GPU 间通信频繁（需高速 NVLink）；PP 减少显存但存在流水线气泡（Pipeline Bubble）。
- **ZeRO-3 的通信开销**：ZeRO-3 大幅节省显存，但每次前向/反向传播都需要跨 GPU 通信参数，增加网络延迟。

## 6) 如何确定最优解
对于 100B 模型在 1024 卡上训练，最优解为 **3D 并行（DP + TP + PP） + ZeRO-3 优化器状态分片**。TP 度设为 8（利用 NVLink 高速互联），PP 度设为 16，DP 度为 8。

## 7) 名词和缩写解释
- **DP（Data Parallel，数据并行）**：多个 GPU 持有相同模型，处理不同数据子集。
- **TP（Tensor Parallel，张量并行）**：将模型权重矩阵切分，由多个 GPU 共同计算。
- **PP（Pipeline Parallel，流水线并行）**：将模型的不同层分配给不同 GPU，形成数据流水线。
- **ZeRO（Zero Redundancy Optimizer）**：微软 DeepSpeed 提出的显存优化技术，通过分片消除优化器、梯度和参数的冗余。
- **TFLOPs（Tera Floating-point Operations Per Second）**：每秒万亿次浮点运算，衡量 AI 芯片算力。
- **NVLink**：NVIDIA 提供的高带宽 GPU 间互联技术。

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
