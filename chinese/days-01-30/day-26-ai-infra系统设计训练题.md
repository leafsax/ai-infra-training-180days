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