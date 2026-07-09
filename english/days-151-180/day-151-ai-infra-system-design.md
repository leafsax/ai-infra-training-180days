# Day 151: Day 151: AI Infra System Design Topic 151

## 1) Topic and Core Examination Areas
**Topic**: Design a high-throughput, low-latency general Large Language Model (LLM) inference service system that supports multi-tenant concurrent requests.
**Core Examination Areas**: Inference service architecture, request scheduling mechanism, batching technology (Batching), optimization of prefill (Prefill) and decode (Decode) phases.

## 2) Requirement Clarification and Metric Definitions
- **qps**: 1000
- **ttft_tp99**: 200ms
- **inter_token_latency_tp99**: 50ms/token
- **tps**: 5000
- **hbm_size**: 80GB HBM
- **context_length**: 32K

## 3) Core Architecture/Technical Component Design
- API Gateway & Request Queue
- Scheduler
- Inference Engine (vLLM, TGI)
- Model Weight Storage

## 4) Deep Dive into Key Technologies and Possible Solutions
- **Static Batching**
- **Continuous Batching/In-flight Batching**

## 5) Trade-off Analysis
- Batch Size增大 vs TTFT和Decode延迟
- Static vs Continuous Batching

## 6) How to Determine the Optimal Solution
Continuous Batching + dynamic KV Cache management

## 7) Full Names and Explanations of All Nouns and Abbreviations
- **LLM**: Large Language Model, large language model
- **QPS**: Queries Per Second
- **TTFT**: Time To First Token
- **TP99**: 99% of request latencies are less than this value
- **TPS**: Tokens Per Second
- **HBM**: High Bandwidth Memory
- **Static Batching**: Static batching
- **Continuous Batching**: Continuous batching
- **Prefill**: Processing input Prompt stage
- **Decode**: Generating output token by token stage
