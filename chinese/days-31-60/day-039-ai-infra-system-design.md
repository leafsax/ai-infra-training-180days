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

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 高级推理服务）**：
  ```mermaid
  graph TD
      A[负载均衡器] --> B[推理 Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU资源管理器]
      D --> E[Kubernetes / Karpenter]
      B --> F[遥测 DCGM/OpenTelemetry]
  ```

- **数据流图（Data Flow Diagram - 连续批处理）**：
  ```mermaid
  flowchart LR
      A[ incoming 请求] --> B[连续批处理调度器]
      B --> C[KV Cache 查找与更新]
      C --> D[GPU Tensor Core 计算]
      D --> E[输出 Tokens]
      E --> F[流式响应]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于KV Cache与PagedAttention**：你提出了使用PagedAttention进行KV cache管理。当在不同上下文长度下HBM严重碎片化时会发生什么？在重负载下，你如何处理缓存驱逐策略，而不降低模型准确性或导致幻觉？
- **关于GPU分区（MIG/vGPU）**：对于MIG或vGPU分区，硬性隔离保证是什么？一个MIG分区中的噪音邻居是否仍会通过共享PCIe通道或CPU内存带宽影响另一个分区？如何实时测量和执行每个分区的SLO？
- **关于自动扩缩容与抖动（Thrashing）**：对于基于KEDA的自动扩缩容，启动新的vLLM pod的从零扩缩容延迟（scale-from-zero latency）是多少？当QPS在扩缩容阈值附近快速波动时（例如10秒内+20%然后-20%），如何防止扩缩容抖动？
