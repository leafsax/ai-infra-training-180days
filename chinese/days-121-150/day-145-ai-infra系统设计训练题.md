### Day 145: Checkpoint 压缩与分层存储（Hot/Warm/Cold）
**1) 题目与考察核心**  
设计Checkpoint的压缩与Tiered Storage（分层存储）架构，降低成本。

**2) 需求澄清与指标定义**  
- 存储分层：Hot (NVMe), Warm (S3 Standard), Cold (S3 Glacier)。
- 压缩：Checkpoint压缩率目标 > 50%。

**3) 核心架构/技术组件设计**  
- 压缩算法：ZIP, GZIP, 或专用权重压缩（如FP8量化临时保存）。
- 自动化策略：最近Checkpoint存NVMe/S3 Standard，旧Checkpoint迁移到Glacier。

**4) 关键技术深入与可能解**  
- 量化压缩：保存FP16/BF16为INT8/FP8，恢复时转回。

**5) Trade-off（权衡）分析**  
- 压缩增加CPU开销和恢复时间；分层存储降低成本但Cold层恢复延迟高。

**6) 如何确定最优解**  
热Checkpoint存高速存储，历史Checkpoint压缩后归档至冷存储。

**7) 名词和缩写解释**  
- **Tiered Storage**: 分层存储（Hot/Warm/Cold）。
- **FP8/INT8**: 8位浮点/整数量化格式。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 任务调度）**：
  ```mermaid
  graph TD
      A[用户作业提交] --> B[调度器 K8s/Slurm]
      B --> C[Gang 调度控制器]
      C --> D[资源亲和性检查器]
      D --> E[GPU节点分配]
      E --> F[作业执行]
  ```

- **数据流图（Data Flow Diagram - 调度流程）**：
  ```mermaid
  flowchart LR
      A[作业请求] --> B[队列与优先级排序]
      B --> C[资源可用性检查]
      C --> D[分配 Gang Pods]
      D --> E[启动训练作业]
  ```
