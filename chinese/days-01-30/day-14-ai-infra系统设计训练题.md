## 第14天：多模型与多租户隔离

### 1) 题目与考察核心
**题目**：设计支持多租户（Multi-tenant）和多模型共存的LLM Serving平台，保证SLA与隔离性。
**考察核心**：租户隔离、优先级调度、资源配额（Quota）。

### 2) 需求澄清与指标定义
- **租户A**：高优先级，要求TTFT < 200ms。
- **租户B**：低优先级，批量处理任务，容忍TTFT < 2s。
- **指标**：高优先级请求的P99延迟不受低优先级请求影响（隔离度 > 90%）。

### 3) 核心架构/技术组件设计
- **Queue Isolation**：为不同租户或优先级创建独立的请求队列。
- **Resource Quotas**：基于GPU显存或Batch size限制租户资源使用。

### 4) 关键技术深入与可能解
- *Priority Batching*：在Continuous Batching中，优先调度高优先级请求的Token生成。
- *Namespace Isolation*：在Kubernetes层面为租户分配独立的Node Pool或GPU Device。

### 5) Trade-off（权衡）分析
- *强隔离 vs 资源利用率*：强隔离（独立GPU节点）保证SLA但资源利用率低；共享GPU节点通过队列隔离提高利用率但存在干扰（Noisy Neighbor）。

### 6) 如何确定最优解
- 采用混合策略：核心租户分配专用GPU节点，普通租户共享GPU节点并通过Priority Scheduler和Quota进行软隔离。

### 7) 名词和缩写全称及解释
- **Multi-tenant (多租户)**：多个用户或团队共享同一套基础设施。
- **SLA (Service Level Agreement)**：服务级别协议，定义服务可用性、延迟等指标。
- **Noisy Neighbor**：嘈杂的邻居，指共享资源时某个租户的高负载影响其他租户性能。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 推理服务）**：
  ```mermaid
  graph TD
      A[客户端/用户] --> B[API网关 / 负载均衡器]
      B --> C[vLLM/TGI 推理引擎]
      C --> D[GPU集群 H100/A100]
      C --> E[模型权重存储 NVMe/S3]
      C --> F[KV Cache 管理器]
      D --> G[GPU监控 DCGM]
  ```

- **数据流图（Data Flow Diagram - 推理流程）**：
  ```mermaid
  flowchart LR
      A[用户提示词] --> B[API网关]
      B --> C[请求队列与批处理]
      C --> D[GPU推理计算]
      D --> E[Token生成]
      E --> F[响应流式输出]
  ```
