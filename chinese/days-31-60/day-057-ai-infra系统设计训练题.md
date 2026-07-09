# 第57天：第57天：AI Infra系统设计训练题

## 1) 题目与考察核心
**题目**：设计一个用于训练 100B 参数大语言模型的分布式训练系统。
**考察核心**：分布式训练并行策略（DP/TP/PP）、显存优化技术（ZeRO）、通信优化。

## 2) 需求澄清与指标定义
- **gpu_count**: 1024 张 H100 80GB GPU
- **training_time**: < 30 天
- **tflops_utilization**: > 60%
- **model_parameters**: 100B（1000亿）参数，FP16/BF16 精度

## 3) 核心架构/技术组件设计
- 数据并行（DP）节点集群
- 张量并行（TP）层
- 流水线并行（PP）阶段
- 优化器状态管理

## 4) 关键技术深入与可能解
- **DP（Data Parallel，数据并行）**
- **TP（Tensor Parallel，张量并行）**
- **PP（Pipeline Parallel，流水线并行）**
- **ZeRO（Zero Redundancy Optimizer，零冗余优化器）**

## 5) Trade-off（权衡）分析
- DP vs TP vs PP
- ZeRO-3 的通信开销

## 6) 如何确定最优解
3D 并行（DP + TP + PP） + ZeRO-3 优化器状态分片

## 7) 名词和缩写解释
- **DP**: Data Parallel，数据并行
- **TP**: Tensor Parallel，张量并行
- **PP**: Pipeline Parallel，流水线并行
- **ZeRO**: Zero Redundancy Optimizer
- **TFLOPs**: Tera Floating-point Operations Per Second
- **NVLink**: NVIDIA 提供的高带宽 GPU 间互联技术


### **8) 组件图与数据流图**

- **组件图（Component Diagram - GPU集群管理）**：
  ```mermaid
  graph TD
      A[集群控制器] --> B[GPU节点 H100]
      B --> C[MIG / vGPU 分区器]
      B --> D[RDMA 网络交换机]
      A --> E[自动扩缩容 KEDA]
      E --> B
  ```

- **数据流图（Data Flow Diagram - 扩缩容流程）**：
  ```mermaid
  flowchart LR
      A[流量突增] --> B[指标收集器 DCGM]
      B --> C[Scaler 控制器]
      C --> D[ Provision 新GPU Pods]
      D --> E[路由流量到新Pods]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于KV Cache与PagedAttention**：你提出了使用PagedAttention进行KV cache管理。当在不同上下文长度下HBM严重碎片化时会发生什么？在重负载下，你如何处理缓存驱逐策略，而不降低模型准确性或导致幻觉？
- **关于GPU分区（MIG/vGPU）**：对于MIG或vGPU分区，硬性隔离保证是什么？一个MIG分区中的噪音邻居是否仍会通过共享PCIe通道或CPU内存带宽影响另一个分区？如何实时测量和执行每个分区的SLO？
- **关于自动扩缩容与抖动（Thrashing）**：对于基于KEDA的自动扩缩容，启动新的vLLM pod的从零扩缩容延迟（scale-from-zero latency）是多少？当QPS在扩缩容阈值附近快速波动时（例如10秒内+20%然后-20%），如何防止扩缩容抖动？
