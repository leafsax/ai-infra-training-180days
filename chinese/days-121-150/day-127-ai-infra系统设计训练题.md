### Day 127: 分布式文件系统 for AI (Lustre, GPFS, Ceph)
**1) 题目与考察核心**  
对比Lustre, GPFS, Ceph，为AI集群选择并设计分布式文件系统架构。

**2) 需求澄清与指标定义**  
- 文件规模：百万级大文件（每个文件 1-10 GB，如预训练数据分片）。
- 并发客户端：1,250个计算节点同时读取。
- 吞吐目标：> 40 GB/s 聚合读取带宽。

**3) 核心架构/技术组件设计**  
- **Lustre**: 分离元数据服务器 (MDS) 和对象存储服务器 (OSS)，客户端通过MDT和OST访问。
- **GPFS**: 分布式元数据管理，无单点故障，适合IBM硬件生态。
- **Ceph**: CRUSH算法实现无中心元数据服务器，对象/块/文件统一存储。

**4) 关键技术深入与可能解**  
- Lustre的MDS瓶颈：百万级文件时MDS可能成为瓶颈，需部署Multiple MDS。
- Ceph的PG (Placement Group) 调优：影响并发度和元数据性能。

**5) Trade-off（权衡）分析**  
- Lustre/GPFS：专业HPC文件系统，吞吐极高，但商业许可或部署复杂；Ceph：开源灵活，但高并发大文件吞吐不如Lustre。

**6) 如何确定最优解**  
企业级AI集群首选Lustre或GPFS；若预算有限且具备Ceph运维能力，可选Ceph并调优PG和OSD并发。

**7) 名词和缩写解释**  
- **MDS**: Metadata Server（元数据服务器）。
- **OST**: Object Storage Target（Lustre中的数据存储节点）。
- **MDT**: Metadata Target（Lustre元数据存储）。
- **GPFS**: General Parallel File System。
- **CRUSH**: Controlled Replication Under Scalable Hashing（Ceph的分布式数据分布算法）。
- **PG**: Placement Group（Ceph数据分布逻辑组）。
- **OSD**: Object Storage Daemon（Ceph数据存储进程）。

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
