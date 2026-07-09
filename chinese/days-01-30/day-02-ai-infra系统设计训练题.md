## 第2天：GPU架构与层次结构

### 1) 题目与考察核心
**题目**：在AI Infra中，如何充分利用GPU硬件层次结构（HBM、NVLink、PCIe、NUMA）来优化模型加载与通信？
**考察核心**：理解GPU硬件拓扑，设计高效的模型部署与多卡通信架构。

### 2) 需求澄清与指标定义
- **HBM容量与带宽**：H100 80GB HBM3，带宽约 3.35 TB/s。
- **NVLink带宽**：单卡间NVLink带宽约 900 GB/s（第三代）。
- **PCIe带宽**：PCIe 5.0 x16 带宽约 32 GB/s（双向）。
- **指标**：模型权重加载时间 < 30s，卡间通信开销 < 5%的总训练/推理时间。

### 3) 核心架构/技术组件设计
- **GPU拓扑感知调度**：使用NUMA-aware和GPU拓扑感知的调度器（如Kubernetes Device Plugin + topology-manager）。
- **高速互联网络**：训练/推理集群内部采用InfiniBand (IB) 或 RoCEv2 网络，结合NVSwitch实现全互联。

### 4) 关键技术深入与可能解
- **模型权重加载策略**：
  - *PCIe加载*：从CPU RAM通过PCIe加载到GPU HBM，带宽受限（~32 GB/s），70B模型加载需数秒至数十秒。
  - *NVLink/NVSwitch共享*：通过GPU间高速互联直接拷贝，或采用分布式权重加载（如DeepSpeed Zero-Infinity）。

### 5) Trade-off（权衡）分析
- *PCIe vs NVLink*：PCIe通用但带宽低；NVLink带宽高但仅适用于同节点GPU。跨节点必须依赖IB/RoCE网络（带宽~200 GB/s，但延迟高于NVLink）。

### 6) 如何确定最优解
- 采用分层加载：热权重（如Attention层）优先通过NVLink加载，或使用In-Memory Distributed Checkpoint加载技术（如JAX的ShardMap或PyTorch FSDP的efficient load）。

### 7) 名词和缩写全称及解释
- **HBM (High Bandwidth Memory)**：高带宽内存，GPU显存。
- **NVLink**：NVIDIA开发的GPU间高速互连技术，带宽远超PCIe。
- **NVSwitch**：NVIDIA的GPU交换芯片，用于构建全互联GPU节点（如HGX节点）。
- **PCIe (Peripheral Component Interconnect Express)**：计算机外设高速串行扩展总线标准。
- **NUMA (Non-Uniform Memory Access)**：非统一内存访问架构，CPU访问不同内存区域的延迟不同。

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
