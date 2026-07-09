## 第26天：监控、性能剖析与可观测性（Monitoring, Profiling, Observability）

### 1) 题目与考察核心
**题目**：设计AI Infra系统的监控与性能剖析平台。
**考察核心**：Prometheus, Grafana, eBPF, NvProf, PyTorch Profiler, 分布式追踪。

### 2) 需求澄清与指标定义
- **监控维度**：GPU利用率、HBM使用率、NCCL通信带宽、请求延迟分布。
- **指标**：监控数据延迟 < 5s，性能剖析（Profiling）开销 < 10%。

### 3) 核心架构/技术组件设计
- **Metrics Pipeline**：GPU指标通过dcgm-exporter采集到Prometheus，Grafana可视化。
- **Tracing**：使用OpenTelemetry追踪请求在Router、Scheduler、Engine间的流转。

### 4) 关键技术深入与可能解
- *NvProf / Nsight Systems*：NVIDIA官方性能剖析工具，深入GPU内核级别。
- *eBPF*：用于无侵入式网络和系统调用监控。

### 5) Trade-off（权衡）分析
- *详细Profiling vs 生产开销*：详细Profiling（如每请求剖析）精度极高但严重影响性能；生产环境应采用抽样（Sampling）或基于指标的监控。

### 6) 如何确定最优解
- 生产环境使用Prometheus + dcgm-exporter + Grafana进行实时监控；性能优化阶段使用Nsight Systems或PyTorch Profiler进行抽样剖析。

### 7) 名词和缩写全称及解释
- **Prometheus**：开源监控系统和时间序列数据库。
- **Grafana**：开源数据可视化与分析平台。
- **eBPF (Extended Berkeley Packet Filter)**：Linux内核技术，用于安全有效地执行定制程序，常用于网络监控。
- **Nsight Systems / NvProf**：NVIDIA的性能剖析和调试工具。

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


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于推理批处理与延迟**：你提到了静态批处理与连续批处理。在中途动态拆分批处理时，内核融合（kernel fusion）的确切开销是多少？当长上下文请求（如128K token）到达并迫使为较短请求进行KV cache驱逐时，如何保证TP99延迟？
- **关于分布式训练通信**：在DP/TP/PP中，如何在1024个GPU的集群中处理straggler（慢节点）节点？如果一个GPU慢5%，整个All-Reduce步骤会阻塞吗？对于使用BF16权重和梯度的NCCL All-Reduce，每步的确切通信量是多少？
- **关于内存优化**：你提到了ZeRO和PagedAttention。对于ZeRO-3，如何在优化器步骤期间最小化跨节点通信开销？对于PagedAttention，当HBM发生碎片化时，确切的驱逐策略是什么（LRU vs LFU vs 基于注意力分数）？
