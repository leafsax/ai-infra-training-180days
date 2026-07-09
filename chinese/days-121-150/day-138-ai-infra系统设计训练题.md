### Day 138: RoCEv2 拥塞控制：PFC, ECN, DCQCN
**1) 题目与考察核心**  
设计RoCEv2无损网络的拥塞控制机制，避免PFC storm和吞吐下降。

**2) 需求澄清与指标定义**  
- 网络：800G RoCEv2以太网。
- 目标：丢包率 < 10^-5，避免PFC死锁。

**3) 核心架构/技术组件设计**  
- **PFC (Priority Flow Control)**: 链路层流控，按优先级暂停发送。
- **ECN (Explicit Congestion Notification)**: 网络层拥塞标记，端点减速。
- **DCQCN**: 结合PFC和ECN的拥塞控制算法。

**4) 关键技术深入与可能解**  
- PFC Storm：一个端口触发PFC，导致上游端口也触发PFC，引发全网流控死锁。
- ECN+DCQCN：交换机通过ECN标记数据包，发送端降低发送速率，避免PFC。

**5) Trade-off（权衡）分析**  
- 纯PFC简单但易死锁；ECN+DCQCN复杂但稳定，需交换机和端点共同支持。

**6) 如何确定最优解**  
RoCEv2无损网络推荐启用ECN+DCQCN，合理配置PFC优先级（仅数据流启用PFC，控制流禁用）。

**7) 名词和缩写解释**  
- **DCQCN**: Data Center Quantized Congestion Notification。
- **PFC Storm**: PFC流控风暴。

---



### **8) 组件图与数据流图**

- **组件图（Component Diagram - 任务调度）**：
  ```mermaid
  graph TD
      A[用户作业提交] --> B[调度器 K8s/Slurm]
      B --> C[Gang 调度控制器]
      C --> D[资源亲和性检查器]
      D --> E[GPU节点分配]
      E --> F[作业执行]
  ```

- **数据流图（Data Flow Diagram - 调度流程）**：
  ```mermaid
  flowchart LR
      A[作业请求] --> B[队列与优先级排序]
      B --> C[资源可用性检查]
      C --> D[分配 Gang Pods]
      D --> E[启动训练作业]
  ```
