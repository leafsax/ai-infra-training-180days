# 第1天：LLM推理服务系统基础设计

## 1) 题目与考察核心
**题目**：设计一个高吞吐、低延迟的通用大语言模型（LLM）推理服务系统，支持多租户并发请求。
**考察核心**：推理服务架构、请求调度机制、批处理技术（Batching）、预填充（Prefill）与解码（Decode）阶段的优化。

## 2) 需求澄清与指标定义
- **QPS（Queries Per Second，每秒查询率）**：预估 1000 QPS。
- **延迟指标**：
  - **TTFT（Time To First Token，首字延迟）**：TP99 < 200ms。
  - **生成延迟（Inter-token Latency）**：TP99 < 50ms/token。
- **吞吐量指标**：**TPS（Tokens Per Second，每秒生成Token数）** > 5000 tokens/s。
- **显存大小**：模型权重加载至 **HBM（High Bandwidth Memory，高带宽内存）**，单卡 80GB HBM，支持最大上下文长度 32K。

## 3) 核心架构/技术组件设计
- **API Gateway & 请求队列**：接收外部请求，进行身份验证与速率限制，将请求放入消息队列（如 Kafka 或 Redis Queue）。
- **调度器（Scheduler）**：从队列中拉取请求，根据批处理策略将请求组合成 Batch。
- **推理引擎（Inference Engine）**：如 vLLM、TGI（Text Generation Inference），负责执行模型的前向传播，管理 KV Cache。
- **模型权重存储**：模型参数以 FP16/BF16 格式存储在 GPU 的 HBM 中。

## 4) 关键技术深入与可能解
- **Static Batching（静态批处理）**：在请求到达时，将多个请求静态组合成一个 Batch 进行 Prefill 和 Decode。缺点是如果某个请求生成结束，Batch 必须等待所有请求完成，导致 GPU 空闲。
- **Continuous Batching（连续批处理/In-flight Batching）**：动态地在一个 Batch 中加入新请求，并移除已完成生成的请求。Prefill 和 Decode 阶段分别处理，最大化 GPU 利用率。

## 5) Trade-off（权衡）分析
- **Batch Size 增大**：提高吞吐量（TPS），但会增加 TTFT（因为需要等待更多请求凑齐 Batch）和 Decode 阶段的延迟。
- **Static vs Continuous Batching**：Static Batching 实现简单，但资源利用率低；Continuous Batching 复杂度高（需处理动态 KV Cache 管理），但吞吐量显著提升。

## 6) 如何确定最优解
通过基准测试（Benchmark）确定最佳 Batch Size 和调度策略。对于高并发、长文本场景，选择 **Continuous Batching + 动态 KV Cache 管理** 作为最优解。

## 7) 名词和缩写解释
- **LLM（Large Language Model，大语言模型）**：基于深度学习的大规模文本生成模型。
- **QPS（Queries Per Second）**：每秒处理的查询请求数。
- **TTFT（Time To First Token）**：从发送请求到接收到第一个生成的 Token 的时间，反映系统的响应速度。
- **TP99**：99% 的请求延迟小于该值，用于衡量系统尾延迟。
- **TPS（Tokens Per Second）**：每秒生成的 Token 数量，衡量推理吞吐量。
- **HBM（High Bandwidth Memory）**：高带宽内存，GPU 使用的高速显存类型（如 H100 的 80GB HBM3）。
- **Static Batching**：静态批处理，提前将请求固定组合成 Batch。
- **Continuous Batching**：连续批处理，动态添加和移除 Batch 中的请求。
- **Prefill**：处理输入 Prompt 的阶段，计算并生成初始的 KV Cache。
- **Decode**：逐 Token 生成输出的阶段。

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
