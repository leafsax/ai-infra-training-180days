### Day 56: 生产环境中的模型热更新（Hot Reloads）

**1) 题目与考察核心**
设计支持模型版本热更新的推理服务。考察核心：零停机模型切换、权重加载与请求迁移。

**2) 需求澄清与指标定义**
- **可用性目标**：模型更新期间服务可用性 > 99.99%。
- **更新频率**：每周1-2次模型版本迭代。

**3) 核心架构/技术组件设计**
- 双轨部署（Blue-Green Deployment）：新模型部署在并行环境，验证通过后切换路由。
- 热加载机制：推理引擎支持动态加载新权重而不重启进程。

**4) 关键技术深入与可能解**
- **Blue-Green Deployment**：维护两套相同环境，切换流量实现零停机更新。
- **模型热重载（Hot Reload）**：在运行时替换模型权重与配置，需保证状态一致性。

**5) Trade-off（权衡）分析**
- Blue-Green需双倍资源；Hot Reload实现简单但可能引发显存冲突或状态不一致。

**6) 如何确定最优解**
采用Blue-Green部署配合K8s Service路由切换，确保更新期间请求无丢失且延迟平稳。

**7) 名词和缩写全称及解释**
- **Blue-Green Deployment**：蓝绿部署，通过维护两套环境实现零停机发布。

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
