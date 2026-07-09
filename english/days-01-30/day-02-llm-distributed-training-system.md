# Day 2: LLM Distributed Training System Architecture Design

## 1) Topic and Core Examination Areas
**Topic**: Design a distributed training system for training a 100B parameter Large Language Model.
**Core Examination Areas**: Distributed training parallel strategies (DP/TP/PP), memory optimization technology (ZeRO), communication optimization.

## 2) Requirement Clarification and Metric Definitions
- **Number of GPUs**: 1024 H100 80GB GPUs.
- **Training time**: < 30 days to complete pre-training.
- **Computing efficiency**: **TFLOPs Utilization (Tera Floating-point Operations Per Second utilization)** > 60%.
- **Model parameters**: 100B (100 billion) parameters, FP16/BF16 precision.

## 3) Core Architecture/Technical Component Design
- **Data Parallel (DP) node cluster**: Shard the dataset and distribute it to multiple GPUs.
- **Tensor Parallel (TP) layer**: Shard model weights (such as Attention matrices, MLP matrices) to multiple GPUs.
- **Pipeline Parallel (PP) stage**: Assign different layers of the model to different GPU groups to form a pipeline.
- **Optimizer state management**: Use ZeRO technology to reduce redundant storage of optimizers, gradients, and parameters.

## 4) Deep Dive into Key Technologies and Possible Solutions
- **DP (Data Parallel, data parallel)**: Each GPU has a complete copy of the model, processes different data shards, and finally synchronizes gradients.
- **TP (Tensor Parallel, tensor parallel)**: Slice the multiplication operation of a single large matrix, and multiple GPUs collaborate to complete one forward/backward propagation.
- **PP (Pipeline Parallel, pipeline parallel)**: Slice model layers in order, and different GPUs simultaneously process data layers of different Batches.
- **ZeRO (Zero Redundancy Optimizer, zero redundancy optimizer)**: Proposed by DeepSpeed, divided into ZeRO-1 (optimizer state sharding), ZeRO-2 (gradient sharding), ZeRO-3 (full sharding of parameters, gradients, and optimizer states).

## 5) Trade-off Analysis
- **DP vs TP vs PP**: DP has small communication volume but high memory usage; TP has low memory usage but frequent GPU-to-GPU communication (requires high-speed NVLink); PP reduces memory but has pipeline bubbles (Pipeline Bubble).
- **Communication overhead of ZeRO-3**: ZeRO-3 significantly saves memory, but requires cross-GPU communication of parameters for each forward/backward propagation, increasing network latency.

## 6) How to Determine the Optimal Solution
For a 100B model trained on 1024 cards, the optimal solution is **3D parallel (DP + TP + PP) + ZeRO-3 optimizer state sharding**. TP degree is set to 8 (utilizing high-speed interconnection of NVLink), PP degree is set to 16, and DP degree is 8.

## 7) Full Names and Explanations of All Nouns and Abbreviations
- **DP (Data Parallel, data parallel)**: Multiple GPUs hold the same model and process different data subsets.
- **TP (Tensor Parallel, tensor parallel)**: Shard model weight matrices and have multiple GPUs compute them together.
- **PP (Pipeline Parallel, pipeline parallel)**: Assign different layers of the model to different GPUs to form a data pipeline.
- **ZeRO (Zero Redundancy Optimizer)**: A memory optimization technology proposed by Microsoft's DeepSpeed, which eliminates redundancy of optimizers, gradients, and parameters through sharding.
- **TFLOPs (Tera Floating-point Operations Per Second)**: Tera floating-point operations per second, measuring AI chip computing power.
- **NVLink**: A high-bandwidth GPU interconnection technology provided by NVIDIA.

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


### **9) Interviewer Follow-up Questions (Sharp & Picky)**

- **On Inference Batching & Latency:** You mentioned Static vs. Continuous Batching. What is the exact kernel fusion overhead when dynamically splitting batches mid-generation? How do you guarantee TP99 latency when a long-context request (e.g., 128K tokens) arrives and forces KV cache eviction for shorter requests?
- **On Distributed Training Communication:** In DP/TP/PP, how do you handle straggler nodes in a 1024-GPU cluster? If one GPU is 5% slower, does the entire All-Reduce step block? What is the exact communication volume per step for NCCL All-Reduce with BF16 weights and gradients?
- **On Memory Optimization:** You cited ZeRO and PagedAttention. For ZeRO-3, how do you minimize the cross-node communication overhead during the optimizer step? For PagedAttention, what is the exact eviction policy (LRU vs. LFU vs. Attention-score-based) when HBM fragmentation occurs?
