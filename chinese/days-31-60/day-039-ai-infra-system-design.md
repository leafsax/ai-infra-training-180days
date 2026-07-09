### Day 39: GPU集群调度：Kubernetes + GPU Operator + Volcano/Yunikorn

**1) 题目与考察核心**
设计基于Kubernetes的GPU集群调度系统。考察核心：K8s GPU调度插件与批量调度器集成。

**2) 需求澄清与指标定义**
- **集群规模**：100个GPU节点。
- **QPS/任务数**：支持数百个推理Pod同时运行。
- **资源指标**：需支持GPU共享、时间切片（Time-Slicing）与整卡分配。

**3) 核心架构/技术组件设计**
- K8s集群 + NVIDIA GPU Operator（提供GPU设备插件）。
- 批量调度器：Volcano或Yunikorn，支持Gang Scheduling与优先级队列。

**4) 关键技术深入与可能解**
- **NVIDIA Device Plugin**：向K8s暴露GPU资源。
- **Time-Slicing（时间切片）**：将单张GPU虚拟给多个Pod共享，通过时间片轮转执行。
- **Gang Scheduling**：确保一批Pod同时调度成功，避免资源死锁。

**5) Trade-off（权衡）分析**
- 整卡分配性能最好但资源利用率低；Time-Slicing提高利用率但引入上下文切换开销与延迟抖动。

**6) 如何确定最优解**
对于LLM推理Pod（需整卡显存与高吞吐），使用整卡分配与Gang Scheduling；对于小模型或开发测试任务，使用Time-Slicing。

**7) 名词和缩写全称及解释**
- **Kubernetes (K8s)**：开源容器编排平台。
- **GPU Operator**：NVIDIA提供的K8s operator，自动部署GPU驱动、Device Plugin等。
- **Volcano / Yunikorn**：开源批量与AI工作负载调度器，支持Gang Scheduling与优先级。

---