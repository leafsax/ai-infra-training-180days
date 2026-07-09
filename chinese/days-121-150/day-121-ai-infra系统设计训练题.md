# 第121天：第121天：AI Infra系统设计训练题

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
