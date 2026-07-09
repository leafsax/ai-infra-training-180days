## 第3天：模型并行基础（DP, TP, PP, MP）

### 1) 题目与考察核心
**题目**：设计一个支持千亿参数（100B+）LLM的分布式推理/训练系统架构。
**考察核心**：理解模型并行（Model Parallelism, MP）的三大基本范式：数据并行（DP）、张量并行（TP）、流水线并行（PP）。

### 2) 需求澄清与指标定义
- **模型大小**：100B参数，FP16精度下权重约200GB。
- **单卡显存**：80GB HBM，单卡无法容纳完整模型。
- **指标**：训练/推理吞吐最大化，通信开销最小化。

### 3) 核心架构/技术组件设计
- **混合并行策略**：DP + TP + PP 三维并行。例如：8路TP（Tensor Parallelism），8路PP（Pipeline Parallelism），4路DP（Data Parallelism），共256张GPU。

### 4) 关键技术深入与可能解
- *Data Parallelism (DP)*：复制完整模型到多个GPU，输入数据分片。
- *Tensor Parallelism (TP)*：将单层网络的权重矩阵切分到多个GPU（如行切分、列切分）。
- *Pipeline Parallelism (PP)*：将模型的不同层分配给不同的GPU，形成流水线。

### 5) Trade-off（权衡）分析
- *DP vs TP*：DP通信简单（All-Reduce梯度），但需完整模型副本；TP通信频繁（GEMM间的All-Reduce或All-Gather），但单卡显存占用低。
- *PP的Bubble时间*：流水线存在空泡（Bubble），即部分GPU在等待上游梯度或激活时处于空闲。

### 6) 如何确定最优解
- 根据模型大小和GPU数量，优先使用TP切分单层（减少跨节点通信），然后使用PP切分层，最后使用DP提升吞吐。使用Auto-Parallel工具（如PyTorch FSDP, Megatron-LM）自动寻找最优切分。

### 7) 名词和缩写全称及解释
- **MP (Model Parallelism)**：模型并行，将模型切分到多个设备。
- **DP (Data Parallelism)**：数据并行，多个GPU持有模型完整副本，处理不同数据。
- **TP (Tensor Parallelism)**：张量并行，将张量（矩阵）切分到多个GPU。
- **PP (Pipeline Parallelism)**：流水线并行，将网络层切分到多个GPU。

---