### Day 126: AI训练数据存储需求与架构
**1) 题目与考察核心**  
设计支撑万卡LLM预训练的存储架构，满足高并发、高吞吐的数据读取需求。

**2) 需求澄清与指标定义**  
- 数据规模：100 TB原始文本数据，处理后Token数据集约 30 TB。
- 并发读取：1,250个计算节点，每节点需同时读取数据，聚合吞吐需 > 50 GB/s。
- 延迟：数据块（Chunk）读取延迟 < 5 ms。

**3) 核心架构/技术组件设计**  
- 存储类型：分布式并行文件系统（如Lustre, GPFS）或对象存储（S3）+ 本地缓存（NVMe SSD）。
- 架构：HDFS/Lustre元数据服务器（MDS） + 多个Object Servers (OSS) + 计算节点本地NVMe缓存。

**4) 关键技术深入与可能解**  
- **Lustre**: 企业级并行文件系统，高吞吐，适合HPC/AI。
- **Ceph**: 分布式对象/块/文件系统，开源，但吞吐和延迟不如Lustre。
- **本地NVMe缓存**: 节点本地SSD缓存热点数据，减少远端存储压力。

**5) Trade-off（权衡）分析**  
- 并行文件系统 vs 对象存储：Lustre/GPFS吞吐高但部署复杂、成本高；S3对象存储成本低但高并发小文件读取延迟高、吞吐受限。

**6) 如何确定最优解**  
对于TB级预训练数据，采用Lustre或GPFS作为主存储，计算节点挂载本地NVMe作为缓存层（Data Cache），通过预取（Prefetch）和异步读取掩盖I/O延迟。

**7) 名词和缩写解释**  
- **LLM**: Large Language Model（大语言模型）。
- **Lustre**: 开源并行文件系统，广泛用于HPC。
- **GPFS**: IBM General Parallel File System（现名IBM Spectrum Scale）。
- **S3**: Amazon Simple Storage Service（对象存储标准）。
- **MDS**: Metadata Server（元数据服务器）。
- **OSS**: Object Storage Server（对象存储服务器）。

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
