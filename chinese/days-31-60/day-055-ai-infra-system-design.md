### Day 55: GPU网络拓扑与互联技术：NVLink, InfiniBand, RoCE

**1) 题目与考察核心**
设计高性能GPU集群网络架构。考察核心：GPU间互联（NVLink）与集群网络（InfiniBand/RoCE）对分布式推理的影响。

**2) 需求澄清与指标定义**
- **集群规模**：64个GPU节点，需支持TP/PP通信。
- **带宽需求**：节点内GPU间带宽 > 900GB/s（NVLink），节点间带宽 > 400Gbps。

**3) 核心架构/技术组件设计**
- 节点内互联：NVLink + NVSwitch实现GPU间高速通信。
- 节点间互联：InfiniBand或RoCE v2网络。

**4) 关键技术深入与可能解**
- **NVLink**：NVIDIA GPU间直接高速互联，带宽远超PCIe。
- **InfiniBand (IB)**：低延迟、高带宽的专用数据中心网络。
- **RoCE (RDMA over Converged Ethernet)**：基于以太网的RDMA技术，降低成本。

**5) Trade-off（权衡）分析**
- InfiniBand性能最优但成本高、生态封闭；RoCE基于以太网更灵活但需精细调优以避免拥塞。

**6) 如何确定最优解**
对于大规模TP/PP推理集群，优先选择NVLink + InfiniBand组合；若成本受限，采用NVLink + 200/400G RoCE v2。

**7) 名词和缩写全称及解释**
- **NVLink / NVSwitch**：NVIDIA GPU间高速互连技术与交换芯片。
- **RDMA (Remote Direct Memory Access)**：允许一台计算机的内存直接访问另一台计算机的内存，绕过CPU。
- **RoCE (RDMA over Converged Ethernet)**：在以太网上实现RDMA的技术。

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
