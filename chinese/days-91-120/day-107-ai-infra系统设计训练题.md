### Day 107: Multimodal Data Parallelism vs Model Parallelism for VLMs

#### 1) 题目与考察核心
设计VLM训练中的数据并行（DP）与模型并行（TP/PP）策略。

#### 2) 需求澄清与指标定义
- **模型规模**：30B参数。
- **GPU集群**：64张H100 (80GB)。
- **显存占用目标**：每GPU显存使用 ≤ 75GB。

#### 3) 核心架构/技术组件设计
- **DP (Data Parallelism)**：数据分片到多个GPU，每个GPU有完整模型副本。
- **TP (Tensor Parallelism)**：将矩阵乘法分片到多个GPU。
- **PP (Pipeline Parallelism)**：将模型层分片到不同GPU组。

#### 4) 关键技术深入与可能解（对比分析不同方案）
- **DP vs TP/PP**：
  - *DP*：简单，但显存随模型大小线性增长。
  - *TP/PP*：降低单GPU显存，但增加通信开销。

#### 5) Trade-off（权衡）分析
- **显存效率 vs 通信开销**：模型并行节省显存但增加NCCL通信。

#### 6) 如何确定最优解
采用ZeRO-DP + TP（张量并行）+ PP（管道并行）的混合并行策略，如3D并行。

#### 7) 名词和缩写全称及解释
- **DP (Data Parallelism)**：数据并行。
- **TP (Tensor Parallelism)**：张量并行，分片矩阵计算。
- **PP (Pipeline Parallelism)**：管道并行，分片模型层。

---

