# Day 1: LLM Inference Service System Design

## 1) Topic and Core Examination Areas
**Topic**: Design a high-throughput, low-latency general Large Language Model (LLM) inference service system that supports multi-tenant concurrent requests.
**Core Examination Areas**: Inference service architecture, request scheduling mechanism, batching technology (Batching), optimization of prefill (Prefill) and decode (Decode) phases.

## 2) Requirement Clarification and Metric Definitions
- **QPS (Queries Per Second, queries per second)**: Estimated 1000 QPS.
- **Latency metrics**:
  - **TTFT (Time To First Token, first token latency)**: TP99 < 200ms.
  - **Generation latency (Inter-token Latency)**: TP99 < 50ms/token.
- **Throughput metrics**: **TPS (Tokens Per Second, tokens generated per second)** > 5000 tokens/s.
- **Memory size**: Model weights loaded into **HBM (High Bandwidth Memory, high-bandwidth memory)**, single card 80GB HBM, supporting maximum context length of 32K.

## 3) Core Architecture/Technical Component Design
- **API Gateway & Request Queue**: Receive external requests, perform authentication and rate limiting, and place requests into a message queue (such as Kafka or Redis Queue).
- **Scheduler**: Pull requests from the queue and combine them into a Batch according to the batching strategy.
- **Inference Engine**: Such as vLLM, TGI (Text Generation Inference), responsible for executing the forward propagation of the model and managing KV Cache.
- **Model Weight Storage**: Model parameters are stored in the GPU's HBM in FP16/BF16 format.

## 4) Deep Dive into Key Technologies and Possible Solutions
- **Static Batching (Static Batching)**: When requests arrive, statically combine multiple requests into a Batch for Prefill and Decode. The disadvantage is that if a request generation ends, the Batch must wait for all requests to complete, causing GPU idle time.
- **Continuous Batching (Continuous Batching/In-flight Batching)**: Dynamically add new requests to a Batch and remove requests that have completed generation. Prefill and Decode phases are processed separately to maximize GPU utilization.

## 5) Trade-off Analysis
- **Increasing Batch Size**: Improves throughput (TPS), but increases TTFT (because it needs to wait for more requests to gather a Batch) and latency in the Decode phase.
- **Static vs Continuous Batching**: Static Batching is simple to implement but has low resource utilization; Continuous Batching has high complexity (requires dynamic KV Cache management) but significantly improves throughput.

## 6) How to Determine the Optimal Solution
Determine the optimal Batch Size and scheduling strategy through benchmarking (Benchmark). For high-concurrency, long-text scenarios, choose **Continuous Batching + dynamic KV Cache management** as the optimal solution.

## 7) Full Names and Explanations of All Nouns and Abbreviations
- **LLM (Large Language Model, large language model)**: A large-scale text generation model based on deep learning.
- **QPS (Queries Per Second)**: The number of query requests processed per second.
- **TTFT (Time To First Token)**: The time from sending a request to receiving the first generated Token, reflecting the response speed of the system.
- **TP99**: 99% of request latencies are less than this value, used to measure system tail latency.
- **TPS (Tokens Per Second)**: The number of Tokens generated per second, measuring inference throughput.
- **HBM (High Bandwidth Memory)**: High-bandwidth memory, the high-speed memory type used by GPUs (such as 80GB HBM3 of H100).
- **Static Batching**: Static batching, which fixes requests into a Batch in advance.
- **Continuous Batching**: Continuous batching, dynamically adding and removing requests in a Batch.
- **Prefill**: The stage of processing the input Prompt, calculating and generating the initial KV Cache.
- **Decode**: The stage of generating output token by token.

### **8) Component Diagram & Data Flow Diagram**

- **Component Diagram**:
  ```mermaid
  graph TD
      A[Client/User] --> B[API Gateway / Load Balancer]
      B --> C[vLLM/TGI Inference Engine]
      C --> D[GPU Cluster H100/A100]
      C --> E[Model Weights Storage NVMe/S3]
      C --> F[KV Cache Manager]
      D --> G[GPU Monitoring DCGM]
  ```

- **Data Flow Diagram**:
  ```mermaid
  flowchart LR
      A[User Prompt] --> B[API Gateway]
      B --> C[Request Queue & Batching]
      C --> D[GPU Inference Compute]
      D --> E[Token Generation]
      E --> F[Response Stream]
  ```
