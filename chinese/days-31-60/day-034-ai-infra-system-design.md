### Day 34: 推理时的模型并行技术：TP, PP, DP

**1) 题目与考察核心**
设计超大模型（如70B+参数）的推理服务。考察核心：Tensor Parallelism (TP), Pipeline Parallelism (PP), Data Parallelism (DP) 在推理中的应用。

**2) 需求澄清与指标定义**
- **模型规模**：70B参数模型。
- **QPS**：500。
- **延迟指标**：TTFT < 500ms，TP99 < 8秒。
- **显存HBM**：使用8x H100 80GB（单卡显存不足以加载70B模型全精度）。

**3) 核心架构/技术组件设计**
- 模型分片层：使用TP将单层神经网络按列/行切分到多卡；PP将不同层分配到不同卡组；DP复制多个模型副本处理不同数据流。

**4) 关键技术深入与可能解**
- **TP (Tensor Parallelism)**：将大矩阵乘法切分到多GPU，通过All-Reduce通信同步。推理时TP通常设为4或8。
- **PP (Pipeline Parallelism)**：将模型层按顺序划分到不同GPU，存在Pipeline Bubble（气泡）浪费。
- **DP (Data Parallelism)**：多个GPU运行相同模型副本，通过请求路由分发。

**5) Trade-off（权衡）分析**
- TP通信开销大但显存效率高；PP减少单卡显存压力但引入延迟与bubble；DP易于扩展但显存重复占用。

**6) 如何确定最优解**
对于70B模型推理，采用TP=4或TP=8（取决于GPU数量与NVLink带宽），结合DP实现横向扩展。PP在推理中较少使用，因推理是逐token生成，PP的bubble影响更大。

**7) 名词和缩写全称及解释**
- **TP (Tensor Parallelism)**：张量并行，将模型权重或激活张量切分到多个GPU。
- **PP (Pipeline Parallelism)**：流水线并行，将模型不同层分配到不同GPU。
- **DP (Data Parallelism)**：数据并行，多个GPU持有相同模型副本，处理不同请求。
- **All-Reduce**：分布式通信原语，用于聚合多个GPU的计算结果。

---