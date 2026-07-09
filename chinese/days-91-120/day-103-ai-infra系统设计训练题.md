### Day 103: Multimodal Training Architecture: Connector & Encoder-Decoder

#### 1) 题目与考察核心
设计VLM的Connector架构和Encoder-Decoder多模态训练架构。

#### 2) 需求澄清与指标定义
- **Connector参数量**：≤ 1B参数（投影层）。
- **显存占用目标**：Connector训练期间显存增加 ≤ 10%。

#### 3) 核心架构/技术组件设计
- **MLP Projection**：将视觉特征映射到语言模型维度。
- **Q-Former (Querying Transformer)**：如BLIP-2使用，通过query tokens压缩视觉特征。

#### 4) 关键技术深入与可能解（对比分析不同方案）
- **MLP vs Q-Former**：
  - *MLP*：简单高效，但可能无法充分压缩视觉信息。
  - *Q-Former*：有效压缩视觉tokens，但增加训练复杂度。

#### 5) Trade-off（权衡）分析
- **复杂度 vs 特征压缩效率**：Q-Former压缩比高但训练慢。

#### 6) 如何确定最优解
对于30B VLM，采用轻量MLP projection或单层Transformer connector，平衡训练速度与性能。

#### 7) 名词和缩写全称及解释
- **MLP (Multi-Layer Perceptron)**：多层感知机，基础神经网络层。
- **Q-Former (Querying Transformer)**：通过query tokens选择重要视觉特征的Transformer模块。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 检查点机制）**：
  ```mermaid
  graph TD
      A[训练集群 GPUs] --> B[检查点管理器]
      B --> C[共享存储 S3/NFS/Ceph]
      A --> D[模型注册表 MLflow/W&B]
      A --> E[梯度同步 RDMA/NCCL]
  ```

- **数据流图（Data Flow Diagram - 检查点流程）**：
  ```mermaid
  flowchart LR
      A[训练步计算] --> B[状态快照 权重/优化器]
      B --> C[异步检查点写入器]
      C --> D[持久化存储]
      D --> E[模型注册表更新]
  ```
