# 第107天：第107天：AI Infra系统设计训练题

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
