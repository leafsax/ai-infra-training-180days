## 第12天：GPU显存卸载（CPU RAM / NVMe Offloading）

### 1) 题目与考察核心
**题目**：当GPU显存不足以加载大模型时，设计显存卸载架构以支持推理。
**考察核心**：CPU Offloading, NVMe Offloading, 动态卸载策略。

### 2) 需求澄清与指标定义
- **模型大小**：70B FP16 = 140GB，单卡80GB GPU。
- **指标**：启用Offloading后，TTFT增加不超过3倍，吞吐下降不超过50%。

### 3) 核心架构/技术组件设计
- **Offload Engine**：在计算前将权重从CPU RAM或NVMe加载到GPU HBM，计算后卸载回。
- **Layer-wise Offloading**：按Layer逐步卸载和加载。

### 4) 关键技术深入与可能解
- *CPU Offloading*：通过PCIe传输，带宽~32 GB/s，延迟高。
- *NVMe Offloading*：使用高速NVMe SSD（如PCIe 4.0 x4, ~7 GB/s），比CPU RAM更慢，但容量大。

### 5) Trade-off（权衡）分析
- *Offloading vs OOM*：Offloading允许运行超大模型，但PCIe/NVMe带宽成为瓶颈，导致TTFT和每Token延迟显著增加。

### 6) 如何确定最优解
- 对于推理，优先使用模型量化（INT4/FP8）或Tensor Parallelism跨多卡。仅在无足够GPU显存时启用Offloading（如vLLM的`--offload-size`）。

### 7) 名词和缩写全称及解释
- **Offloading (卸载)**：将数据或计算从GPU显存转移到CPU内存或磁盘，以节省显存。
- **NVMe (Non-Volatile Memory express)**：非易失性内存主机控制器接口，高速SSD标准。

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
