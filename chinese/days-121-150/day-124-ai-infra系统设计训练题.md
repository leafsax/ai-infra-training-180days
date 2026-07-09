### Day 124: NVLink, NVSwitch 与 GPU 间互连
**1) 题目与考察核心**  
设计单机多卡（如8x H100）GPU间互连架构，最大化节点内通信带宽。

**2) 需求澄清与指标定义**  
- 节点配置：8x H100 GPU，2x CPU (AMD/Intel)。
- 节点内带宽：NVLink 4.0，双向带宽 900 GB/s per GPU。
- 延迟：GPU间通信延迟 < 1 μs。

**3) 核心架构/技术组件设计**  
- 全互连拓扑：通过NVSwitch构建全连接（Full Mesh）或近全连接拓扑，确保任意两GPU间带宽达到900 GB/s。
- 软件栈：CUDA Toolkit, NCCL配置使用NVLink而非PCIe。

**4) 关键技术深入与可能解**  
- **NVLink vs PCIe**: NVLink带宽远高于PCIe Gen5（~32 GB/s双向），延迟更低。
- **NVSwitch**: 允许多个GPU在节点内形成高速交换网络，避免PCIe总线瓶颈。

**5) Trade-off（权衡）分析**  
- NVSwitch增加节点成本和功耗，但极大提升分布式训练效率（特别是TP/PP并行时）。若使用低端服务器无NVSwitch，则GPU间通信被迫走PCIe，导致All-Reduce瓶颈。

**6) 如何确定最优解**  
LLM训练必须采用支持NVSwitch的GPU服务器（如NVIDIA DGX H100），并确保NCCL配置识别NVLink拓扑（`NCCL_DEBUG=INFO`）。

**7) 名词和缩写解释**  
- **NVLink**: NVIDIA GPU间高速互连总线。
- **NVSwitch**: NVIDIA基于NVLink的交换机芯片，实现多GPU全互连。
- **PCIe**: Peripheral Component Interconnect Express。
- **TP**: Tensor Parallelism（张量并行）。
- **PP**: Pipeline Parallelism（流水线并行）。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 网络架构）**：
  ```mermaid
  graph TD
      A[作业调度器 Slurm/K8s] --> B[计算节点 GPU+CPU]
      B --> C[RDMA网络 InfiniBand/RoCE]
      B --> D[分布式存储 Lustre/GPFS]
      C --> E[All-Reduce通信 NCCL]
  ```

- **数据流图（Data Flow Diagram - 训练数据与计算）**：
  ```mermaid
  flowchart LR
      A[数据加载器] --> B[本地 NVMe 缓存]
      B --> C[GPU计算内核]
      C --> D[NCCL All-Reduce via RDMA]
      D --> C
      C --> E[梯度更新]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于网络可靠性与PFC风暴**：对于带有DCQCN的RoCEv2或InfiniBand，当发生拥塞时，如何防止PFC（优先级流控制）风暴？当800G NIC或交换链路意外flap时，确切的恢复时间和数据丢失场景是什么？
- **关于分布式存储I/O**：对于Lustre/GPFS/Ceph等分布式文件系统，如何处理多个GPU节点同时产生的小随机I/O模式？元数据服务器（metadata server）的瓶颈是什么，如何在不引入单点故障的情况下进行扩展？
- **关于Gang Scheduling与抢占**：对于K8s/Slurm中的Gang Scheduling，如果所需gang中的一个节点不健康或被驱逐，会发生什么？如何在不同租户队列之间处理抢占公平性，以防止长期运行的训练作业被饥饿？
