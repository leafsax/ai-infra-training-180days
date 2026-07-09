### Day 149: GPU 分区（MIG, vGPU）用于多租户 AI 集群
**1) 题目与考察核心**  
设计GPU分区技术（MIG, vGPU）以实现多租户资源隔离与利用率提升。

**2) 需求澄清与指标定义**  
- 目标：将H100/A100 GPU划分为多个实例，供不同小作业使用。
- 分区粒度：MIG支持最多7个实例（A100）。

**3) 核心架构/技术组件设计**  
- **MIG (Multi-Instance GPU)**: NVIDIA A100/H100硬件级GPU分区。
- **vGPU**: 虚拟化GPU，软件定义资源分配。

**4) 关键技术深入与可能解**  
- MIG：硬隔离，安全但固定粒度；vGPU：灵活但性能有开销。

**5) Trade-off（权衡）分析**  
- MIG适合多租户隔离但降低单作业大GPU利用率；vGPU灵活但需驱动支持。

**6) 如何确定最优解**  
大模型训练使用完整GPU；推理或小实验作业使用MIG/vGPU。

**7) 名词和缩写解释**  
- **MIG**: Multi-Instance GPU。
- **vGPU**: Virtual GPU。

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
