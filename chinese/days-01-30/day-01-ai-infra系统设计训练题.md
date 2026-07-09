## 第1天：LLM Serving基础与核心指标

### 1) 题目与考察核心
**题目**：设计一个面向公网的通用大语言模型（LLM）推理服务系统，要求支持高并发文本生成请求。
**考察核心**：理解LLM Serving的核心性能指标（QPS、TTFT、TP99延迟、TPS等），并建立系统设计的初步指标框架。

### 2) 需求澄清与指标定义
- **QPS (Queries Per Second)**：每秒查询率。预估目标QPS为 1000 QPS。
- **TTFT (Time To First Token)**：首个Token生成时间。要求冷启动TTFT < 500ms，热启动TTFT < 200ms。
- **TP99延迟**：99%的请求生成完整响应的耗时。要求TP99 < 3s（假设平均响应长度为1024个Token，生成速度需达到TPS > 340 Tokens/s）。
- **TPS (Tokens Per Second)**：每秒生成的Token数。系统整体吞吐需达到 300,000 Tokens/s。
- **显存大小 (HBM)**：假设使用Llama-3-70B模型，FP16精度下模型权重约140GB，需配置8x HBM40GB或8x HBM80GB的GPU（如H100 80GB）。

### 3) 核心架构/技术组件设计
- **API Gateway**：处理HTTP/gRPC请求，进行身份验证、限流与路由。
- **Request Scheduler**：调度器，负责将请求放入队列，并根据Batching策略组合请求。
- **Model Engine**：底层推理引擎（如vLLM, TensorRT-LLM），负责计算图执行与KV Cache管理。
- **GPU Cluster**：计算节点集群，部署模型权重与运行时状态。

### 4) 关键技术深入与可能解
- **请求处理模式**：
  - *Request-by-Request*：每次只处理一个请求，延迟低但吞吐极低。
  - *Batching*：将多个请求组合成一个Batch进行并行计算，提升GPU利用率。

### 5) Trade-off（权衡）分析
- **低延迟 vs 高吞吐**：小Batch size降低TTFT但降低GPU吞吐；大Batch size提高吞吐但增加排队延迟，导致TTFT上升。

### 6) 如何确定最优解
- 通过压测（Load Testing）绘制“延迟-吞吐”曲线（Latency-Throughput Curve），找到系统Pareto最优的Batch size和调度策略。

### 7) 名词和缩写全称及解释
- **QPS (Queries Per Second)**：每秒查询率，衡量系统每秒处理的请求数量。
- **TTFT (Time To First Token)**：首个Token输出时间，衡量用户感受到响应速度的关键指标。
- **TP99 (99th Percentile Latency)**：99%的请求延迟小于该值，用于衡量长尾延迟。
- **TPS (Tokens Per Second)**：每秒生成的Token数量，衡量LLM生成的吞吐量。
- **HBM (High Bandwidth Memory)**：高带宽内存，GPU专用的显存类型（如HBM2e, HBM3），提供远超GDDR的带宽。

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
