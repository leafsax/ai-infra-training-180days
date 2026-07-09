# 第58天：第58天：AI Infra系统设计训练题

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

- **组件图（Component Diagram - 分布式训练）**：
  ```mermaid
  graph TD
      A[数据加载器] --> B[训练节点 GPU+CPU]
      B --> C[NCCL All-Reduce 网络]
      B --> D[检查点存储 S3/NFS]
      B --> E[模型注册表 MLflow]
      C --> B
  ```

- **数据流图（Data Flow Diagram - 训练流程）**：
  ```mermaid
  flowchart LR
      A[训练数据] --> B[各GPU梯度计算]
      B --> C[NCCL梯度同步]
      C --> D[权重更新]
      D --> E[检查点保存]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于流量拆分与路由**：你提到了用于流量路由的特性标志（feature flags）和服务网格（service mesh）。基于百分比的流量拆分的精确算法是什么？如何在canary发布期间确保粘性会话或基于用户的特定路由，而不引入显著的延迟开销？
- **关于统计显著性与A/B测试**：对于A/B测试两个模型版本，你如何确定达到统计显著性所需的最小样本量？你跟踪哪些指标以确保canary模型不会降低用户参与度或增加错误率？
- **关于回滚与影子部署**：在影子部署中，你将实时流量路由到旧模型和新模型。你如何处理应该仅由稳定模型触发的有状态操作或副作用（例如日志记录、分析）？如果canary模型表现出严重问题，零停机时间回滚的确切RTO（恢复时间目标）是多少？
