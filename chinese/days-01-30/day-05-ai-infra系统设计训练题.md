## 第5天：张量并行（Tensor Parallelism）深入

### 1) 题目与考察核心
**题目**：设计并实现一个基于Megatron-LM的张量并行（TP）推理/训练引擎，支持70B模型在8卡节点内的切分。
**考察核心**：理解TP中的列切分（Column-wise）、行切分（Row-wise）及Attention层的TP实现。

### 2) 需求澄清与指标定义
- **TP Degree**：8路张量并行（TP=8），适用于单节点8卡（如HGX H100）。
- **通信开销**：TP内部的All-Reduce通信延迟需控制在微秒级，带宽利用率 > 80%。

### 3) 核心架构/技术组件设计
- **Megatron-LM TP Core**：实现Linear层的列切分（Q/K/V投影）和行切分（Output投影）。
- **All-Reduce通信池**：使用NCCL库管理TP间的通信流。

### 4) 关键技术深入与可能解
- *Linear层切分*：
  - 列切分（Column-Parallel Linear）：输入相同，权重按列切分，输出需All-Reduce。
  - 行切分（Row-Parallel Linear）：权重按行切分，输入需广播（Broadcast）或复制，输出无需All-Reduce。
- *Attention TP*：Q/K/V投影采用列切分，Attention输出采用行切分。

### 5) Trade-off（权衡）分析
- *TP粒度*：TP=8通信频繁但单卡显存低；TP=1显存高但无法运行大模型。TP度越大，All-Reduce通信开销呈线性增加。

### 6) 如何确定最优解
- 通常TP度等于节点内GPU数量（如8或4），以利用NVLink全互联拓扑，最小化跨节点通信。

### 7) 名词和缩写全称及解释
- **All-Reduce**：分布式通信原语，所有GPU贡献数据，求和后分发到所有GPU。
- **NCCL (NVIDIA Collective Communications Library)**：NVIDIA提供的分布式通信库，支持All-Reduce, All-Gather等。
- **Column-Parallel / Row-Parallel Linear**：张量并行中线性层的两种切分方式。

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
