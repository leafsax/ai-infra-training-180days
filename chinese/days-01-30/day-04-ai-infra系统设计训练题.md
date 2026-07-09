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
