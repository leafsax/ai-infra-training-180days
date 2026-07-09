# 第31天：第31天：AI Infra系统设计训练题

## 1) 题目与考察核心
**题目**：设计一个高吞吐、低延迟的通用大语言模型（LLM）推理服务系统，支持多租户并发请求。
**考察核心**：推理服务架构、请求调度机制、批处理技术（Batching）、预填充（Prefill）与解码（Decode）阶段的优化。

## 2) 需求澄清与指标定义
- **qps**: 1000
- **ttft_tp99**: 200ms
- **inter_token_latency_tp99**: 50ms/token
- **tps**: 5000
- **hbm_size**: 80GB HBM
- **context_length**: 32K

## 3) 核心架构/技术组件设计
- API Gateway & 请求队列
- 调度器（Scheduler）
- 推理引擎（Inference Engine）
- 模型权重存储

## 4) 关键技术深入与可能解
- **Static Batching（静态批处理）**
- **Continuous Batching（连续批处理/In-flight Batching）**

## 5) Trade-off（权衡）分析
- Batch Size 增大 vs TTFT和Decode延迟
- Static vs Continuous Batching

## 6) 如何确定最优解
Continuous Batching + 动态 KV Cache 管理

## 7) 名词和缩写解释
- **LLM**: Large Language Model，大语言模型
- **QPS**: Queries Per Second
- **TTFT**: Time To First Token
- **TP99**: 99%的请求延迟小于该值
- **TPS**: Tokens Per Second
- **HBM**: High Bandwidth Memory
- **Static Batching**: 静态批处理
- **Continuous Batching**: 连续批处理
- **Prefill**: 处理输入Prompt阶段
- **Decode**: 逐Token生成输出阶段


### **8) 组件图与数据流图**

- **组件图（Component Diagram - 高级推理服务）**：
  ```mermaid
  graph TD
      A[负载均衡器] --> B[推理 Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU资源管理器]
      D --> E[Kubernetes / Karpenter]
      B --> F[遥测 DCGM/OpenTelemetry]
  ```

- **数据流图（Data Flow Diagram - 连续批处理）**：
  ```mermaid
  flowchart LR
      A[ incoming 请求] --> B[连续批处理调度器]
      B --> C[KV Cache 查找与更新]
      C --> D[GPU Tensor Core 计算]
      D --> E[输出 Tokens]
      E --> F[流式响应]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于KV Cache与PagedAttention**：你提出了使用PagedAttention进行KV cache管理。当在不同上下文长度下HBM严重碎片化时会发生什么？在重负载下，你如何处理缓存驱逐策略，而不降低模型准确性或导致幻觉？
- **关于GPU分区（MIG/vGPU）**：对于MIG或vGPU分区，硬性隔离保证是什么？一个MIG分区中的噪音邻居是否仍会通过共享PCIe通道或CPU内存带宽影响另一个分区？如何实时测量和执行每个分区的SLO？
- **关于自动扩缩容与抖动（Thrashing）**：对于基于KEDA的自动扩缩容，启动新的vLLM pod的从零扩缩容延迟（scale-from-zero latency）是多少？当QPS在扩缩容阈值附近快速波动时（例如10秒内+20%然后-20%），如何防止扩缩容抖动？
