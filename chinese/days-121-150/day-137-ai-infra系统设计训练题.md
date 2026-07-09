### Day 137: UCX (Unified Communication X) 与 AI 工作负载
**1) 题目与考察核心**  
设计基于UCX的通用通信栈，支持RDMA、TCP、SHM等多种传输协议。

**2) 需求澄清与指标定义**  
- 目标：统一GPU间（SHM）、节点内（NVLink）、节点间（RDMA/TCP）的通信接口。
- 性能：UCX底层传输自动选择最优路径。

**3) 核心架构/技术组件设计**  
- UCX组件：`ucp` (UCX Process), `uct` (UCX Transport), `ucm` (UCX Memory)。
- 传输优先级：SHM > NVLink > RDMA > TCP。

**4) 关键技术深入与可能解**  
- UCX的Transport Selection：根据拓扑自动选择SHM（进程间）、ib_rc（InfiniBand可靠连接）、rc_uc（RoCEv2）等。

**5) Trade-off（权衡）分析**  
- UCX提供灵活性但增加栈深度；原生NCCL/ MPI更专一但跨平台兼容性弱。

**6) 如何确定最优解**  
AI集群中，NCCL底层已集成UCX或自研RDMA栈；自定义通信应用推荐使用UCX。

**7) 名词和缩写解释**  
- **UCX**: Unified Communication X。
- **SHM**: Shared Memory（共享内存）。
- **ib_rc**: InfiniBand Reliable Connection。

---

