## 第18天：Checkpointing与恢复机制

### 1) 题目与考察核心
**题目**：设计千亿参数模型的分布式Checkpoint与恢复系统。
**考察核心**：Checkpoint格式（Sharded vs Unified），异步保存，存储选型。

### 2) 需求澄清与指标定义
- **Checkpoint大小**：70B模型FP16约140GB，加上优化器状态可达500GB+。
- **指标**：Checkpoint保存时间 < 5分钟，恢复时间 < 3分钟。

### 3) 核心架构/技术组件设计
- **Sharded Checkpointing**：每个GPU保存自己分片的权重，最后合并或分布式加载。
- **Storage Backend**：使用高性能分布式文件系统（Ceph, NFS）或对象存储（S3）+ 缓存层。

### 4) 关键技术深入与可能解
- *Unified Checkpoint*：所有GPU将分片写入一个统一格式文件，便于跨框架加载（如Hugging Face format）。
- *Async Checkpointing*：在后台线程保存Checkpoint，不阻塞训练步。

### 5) Trade-off（权衡）分析
- *Sharded vs Unified*：Sharded保存快但跨框架迁移麻烦；Unified格式标准但合并开销大。

### 6) 如何确定最优解
- 训练期间使用Sharded Checkpoint以最小化停顿，定期转换为Unified格式用于模型发布和跨框架推理。

### 7) 名词和缩写全称及解释
- **Checkpoint (检查点)**：训练过程中保存的模型状态（权重、优化器状态、步数），用于恢复或推理。
- **Sharded Checkpoint**：分片检查点，每个设备保存部分状态。

---

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
