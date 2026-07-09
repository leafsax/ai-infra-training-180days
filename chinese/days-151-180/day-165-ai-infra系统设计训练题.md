### Day 165: 网络成本优化：集群内外网络设计 (Network Cost Optimization)

**1) 题目与考察核心**
优化AI集群内部与外部网络成本与性能。考察核心：Intra-cluster网络（RoCEv2/InfiniBand）、Inter-cluster网络设计。

**2) 需求澄清与指标定义**
- **业务场景**：万卡GPU集群，需支持分布式训练（DP/TP/PP）。
- **网络带宽目标**：Intra-cluster 网络带宽 ≥ 800 Gbps per node。
- **网络延迟**：Rendezvous 通信延迟 < 10μs。

**3) 核心架构/技术组件设计**
- Intra-cluster网络：InfiniBand或RoCEv2无损网络。
- Inter-cluster网络：高带宽以太网用于跨机房训练或备份。
- 网络流量工程：拓扑感知路由优化。

**4) 关键技术深入与可能解**
- **InfiniBand (IB)** vs **RoCEv2 (RDMA over Converged Ethernet)**：IB性能高但成本高且生态封闭；RoCEv2基于以太网，成本较低且生态开放，需无损网络配置。
- **拓扑感知调度** vs **随机调度**：拓扑感知减少跨交换机通信，降低网络拥塞但调度复杂。

**5) Trade-off（权衡）分析**
- 性能 vs 成本：InfiniBand性能最优但成本极高；RoCEv2性价比高但需精细调优。
- 调度复杂度 vs 网络效率：拓扑感知提升效率但增加调度器负担。

**6) 如何确定最优解**
采用RoCEv2无损网络 + 拓扑感知调度，平衡性能与成本，满足万卡集群训练需求。

**7) 名词和缩写解释**
- **InfiniBand (IB)**：高性能网络协议，低延迟高带宽，常用于AI集群。
- **RoCEv2 (RDMA over Converged Ethernet version 2)**：基于以太网的RDMA技术，提供接近InfiniBand的性能。
- **RDMA (Remote Direct Memory Access)**：允许一台计算机的内存直接访问另一台计算机的内存，无需CPU介入。
- **DP/TP/PP**：数据并行(Data Parallel)、张量并行(Tensor Parallel)、流水线并行(Pipeline Parallel)。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 安全与合规）**：
  ```mermaid
  graph TD
      A[客户端请求] --> B[API网关 TLS/认证]
      B --> C[数据清洗 PII脱敏]
      C --> D[安全模型推理]
      D --> E[审计与合规日志]
      D --> F[FinOps成本追踪]
  ```

- **数据流图（Data Flow Diagram - 安全推理）**：
  ```mermaid
  flowchart LR
      A[用户输入] --> B[加密与认证]
      B --> C[PII检测与脱敏]
      C --> D[模型推理引擎]
      D --> E[输出验证]
      E --> F[记录审计与指标]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于PII脱敏与误报率**：对于你的PII检测和脱敏模块，可接受的误报率是多少？如何确保脱敏后的文本不会破坏模型的上下文窗口，或导致模型生成不安全或无意义的输出？
- **关于Spot实例与抢占**：对于使用spot实例的成本优化，你的训练作业可接受的最大抢占率是多少？当spot实例仅提前2分钟警告终止时，你如何动态迁移或保存检查点？
- **关于量子/后量子加密**：你提到了Kyber和Dilithium用于后量子密码学。将这些算法应用于实时推理流量时，在延迟和带宽方面的开销是多少？如何在不重新设计服务网关的情况下实现密码敏捷性（crypto-agility）？
