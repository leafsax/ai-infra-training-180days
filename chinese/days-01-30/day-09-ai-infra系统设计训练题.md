## 第9天：Batching策略（Static vs Dynamic vs Continuous Batching）

### 1) 题目与考察核心
**题目**：设计LLM推理引擎的调度器，最大化吞吐量同时保证延迟SLA。
**考察核心**：对比Static Batching, Dynamic Batching, Continuous Batching（In-flight Batching）。

### 2) 需求澄清与指标定义
- **请求到达率**：泊松分布，平均间隔 50ms。
- **响应长度分布**：长短不一，平均1024 Token，P99为4096 Token。
- **指标**：吞吐量 > 200k TPS，P99延迟 < 3s。

### 3) 核心架构/技术组件设计
- **Scheduler Component**：维护就绪队列（Ready Queue）和运行中队列（Running Queue）。

### 4) 关键技术深入与可能解
- *Static Batching*：固定Batch size，需等待满Batch或超时，延迟高。
- *Dynamic Batching*：动态收集请求到Batch，直到满Batch或超时，平衡延迟与吞吐。
- *Continuous Batching (In-flight Batching)*：请求生成完成后立即从Batch中移除，新请求立即插入，无等待满Batch的开销。

### 5) Trade-off（权衡）分析
- *Dynamic vs Continuous*：Dynamic实现简单但存在Padding浪费和等待时间；Continuous Batching实现复杂（需支持动态KV Cache调整），但吞吐最高，Padding最少。

### 6) 如何确定最优解
- 现代LLM Serving（如vLLM, SGLang）均采用Continuous Batching架构，结合PagedAttention实现动态KV Cache管理。

### 7) 名词和缩写全称及解释
- **Static Batching**：静态批处理，批次大小固定。
- **Dynamic Batching**：动态批处理，根据到达情况和超时策略动态组合Batch。
- **Continuous Batching (In-flight Batching)**：飞行中批处理，允许在生成过程中动态加入新请求或移除完成请求。

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
