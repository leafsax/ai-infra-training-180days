### Day 40: GPU共享与虚拟化技术：MIG, vGPU, Time-Slicing

**1) 题目与考察核心**
设计支持多租户的GPU资源共享系统。考察核心：MIG、vGPU与Time-Slicing的机制与适用场景。

**2) 需求澄清与指标定义**
- **集群规模**：50个节点，每节点8x A100 80GB。
- **租户数**：10个团队，需隔离推理任务与实验任务。

**3) 核心架构/技术组件设计**
- 虚拟化层：根据任务类型选择MIG（多实例GPU）、vGPU或Time-Slicing。
- 调度层：K8s + 资源配额（Quota）管理。

**4) 关键技术深入与可能解**
- **MIG (Multi-Instance GPU)**：将A100物理GPU划分为多个独立实例（如7个实例各10GB显存），硬件级隔离，性能无损。
- **vGPU (virtual GPU)**：NVIDIA vGPU软件虚拟化，支持图形与计算共享，但需特定许可证。
- **Time-Slicing**：K8s Device Plugin支持的时间片共享，无硬件隔离，存在性能干扰。

**5) Trade-off（权衡）分析**
- MIG提供最强隔离与确定性性能，但划分固定且浪费未使用部分；Time-Slicing灵活但存在噪声邻居（Noisy Neighbor）问题。

**6) 如何确定最优解**
生产级LLM推理使用整卡或MIG划分（若模型小）；多租户实验环境使用Time-Slicing配合QoS限制。

**7) 名词和缩写全称及解释**
- **MIG (Multi-Instance GPU)**：NVIDIA A100/H100支持的硬件级GPU分割技术。
- **vGPU (virtual GPU)**：软件定义的虚拟GPU技术。
- **Noisy Neighbor**：共享资源环境中，其他任务占用资源导致性能下降的现象。

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
