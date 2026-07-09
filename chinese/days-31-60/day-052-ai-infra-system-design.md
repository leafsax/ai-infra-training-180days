### Day 52: Serverless推理平台与冷启动（Cold Start）缓解策略

**1) 题目与考察核心**
设计Serverless LLM推理服务。考察核心：冷启动问题、模型预热与弹性伸缩。

**2) 需求澄清与指标定义**
- **流量特征**：稀疏请求，需支持秒级弹性。
- **冷启动目标**：将首次请求TTFT从10秒降低至2秒内。

**3) 核心架构/技术组件设计**
- 容器镜像层：预加载模型权重至本地NVMe或内存。
- 预热机制（Warm-up）：定期发送dummy请求保持Pod活跃，或维护最小Pod副本。

**4) 关键技术深入与可能解**
- **Cold Start**：新Pod启动、模型加载至GPU显存导致的初始高延迟。
- **Snapshot / Image Warm Pool**：提前创建包含模型权重的容器镜像或GPU快照。

**5) Trade-off（权衡）分析**
- 维持Warm Pool增加闲置成本；完全弹性导致冷启动延迟高。

**6) 如何确定最优解**
结合最小副本数（Min Replicas）与镜像预拉取（Pre-pulling），对低频请求设置合理的冷却时间（Cool-down）以平衡成本与延迟。

**7) 名词和缩写全称及解释**
- **Serverless Inference**：无服务器推理，按需自动分配计算资源，按实际使用计费。
- **Cold Start**：服务实例首次启动或从休眠中唤醒时的延迟。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - GPU集群管理）**：
  ```mermaid
  graph TD
      A[集群控制器] --> B[GPU节点 H100]
      B --> C[MIG / vGPU 分区器]
      B --> D[RDMA 网络交换机]
      A --> E[自动扩缩容 KEDA]
      E --> B
  ```

- **数据流图（Data Flow Diagram - 扩缩容流程）**：
  ```mermaid
  flowchart LR
      A[流量突增] --> B[指标收集器 DCGM]
      B --> C[Scaler 控制器]
      C --> D[ Provision 新GPU Pods]
      D --> E[路由流量到新Pods]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于KV Cache与PagedAttention**：你提出了使用PagedAttention进行KV cache管理。当在不同上下文长度下HBM严重碎片化时会发生什么？在重负载下，你如何处理缓存驱逐策略，而不降低模型准确性或导致幻觉？
- **关于GPU分区（MIG/vGPU）**：对于MIG或vGPU分区，硬性隔离保证是什么？一个MIG分区中的噪音邻居是否仍会通过共享PCIe通道或CPU内存带宽影响另一个分区？如何实时测量和执行每个分区的SLO？
- **关于自动扩缩容与抖动（Thrashing）**：对于基于KEDA的自动扩缩容，启动新的vLLM pod的从零扩缩容延迟（scale-from-zero latency）是多少？当QPS在扩缩容阈值附近快速波动时（例如10秒内+20%然后-20%），如何防止扩缩容抖动？
