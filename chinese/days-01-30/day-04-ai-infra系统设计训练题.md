## 第4天：数据并行与ZeRO优化

### 1) 题目与考察核心
**题目**：在大规模分布式训练（如1024卡）中，如何解决数据并行（DP）带来的显存瓶颈与通信开销？
**考察核心**：深入理解ZeRO（Zero Redundancy Optimizer）优化器及其不同阶段（ZeRO-1/2/3）。

### 2) 需求澄清与指标定义
- **显存占用分布**：训练中显存主要分为模型权重（Weights）、梯度（Gradients）、优化器状态（Optimizer States）。
- **指标**：在1024卡上训练70B模型，要求显存利用率 > 85%，通信带宽利用率 > 70%。

### 3) 核心架构/技术组件设计
- **ZeRO引擎集成**：在PyTorch DDP或DeepSpeed框架中启用ZeRO-3，实现权重、梯度、优化器状态的分布式分片。

### 4) 关键技术深入与可能解
- *ZeRO-1*：分片优化器状态（Optimizer States），减少DP冗余。
- *ZeRO-2*：在ZeRO-1基础上，分片梯度（Gradients）。
- *ZeRO-3*：在ZeRO-2基础上，分片模型权重（Weights），每个GPU只保存部分权重，通过All-Gather动态获取。

### 5) Trade-off（权衡）分析
- *ZeRO-3 vs FSDP*：ZeRO-3通信开销较大（频繁的All-Gather/All-Reduce），但显存节省极致；FSDP（Fully Sharded Data Parallel）是PyTorch原生的ZeRO-3实现，可通过`offload_to_cpu`进一步将非活动参数卸载到CPU RAM。

### 6) 如何确定最优解
- 对于内存受限场景（如70B+模型在A100 40GB上），启用ZeRO-3 + Offload to CPU/NVMe。对于速度优先场景，使用ZeRO-2或FSDP with no offload。

### 7) 名词和缩写全称及解释
- **ZeRO (Zero Redundancy Optimizer)**：微软DeepSpeed提出的显存优化技术，消除DP中的冗余状态。
- **DDP (Distributed Data Parallel)**：PyTorch默认的数据并行分布式训练策略。
- **FSDP (Fully Sharded Data Parallel)**：PyTorch原生的分片数据并行实现，基于ZeRO-3理念。
- **Optimizer States**：优化器状态，如Adam优化器中的动量（momentum）和方差（variance）变量，通常占用显存最大。

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
