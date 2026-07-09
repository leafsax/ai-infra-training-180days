### Day 31: LLM推理服务基础与延迟指标设计

**1) 题目与考察核心**
设计一个低延迟的大语言模型（LLM）推理服务系统。考察核心：理解推理服务的延迟指标（TTFT, TPOT, TP99）及系统架构设计。

**2) 需求澄清与指标定义**
- **QPS（Queries Per Second）**：预估峰值QPS为 1000。
- **延迟指标**：TTFT（Time to First Token）需小于 200ms；TP99延迟（整个响应完成的99分位延迟）需小于 3秒。
- **吞吐量TPS（Tokens Per Second）**：系统需支持总输出吞吐量 5000 tokens/s。
- **显存大小HBM（High Bandwidth Memory）**：使用8x H100 GPU，每卡80GB HBM，总显存640GB。

**3) 核心架构/技术组件设计**
- 请求接入层：API Gateway + 负载均衡器（Load Balancer）。
- 推理服务层：部署推理引擎（如vLLM或TGI），支持GPU资源池化。
- 存储层：模型权重存储在高速NVMe SSD或分布式存储（如Ceph）中，加载至GPU HBM。

**4) 关键技术深入与可能解**
- **静态批处理（Static Batching）**：在推理前将请求固定分组，简单但可能导致GPU空闲。
- **动态批处理（Dynamic Batching）**：请求到达时动态合并，提高GPU利用率，但增加调度复杂度。
- **连续批处理（Continuous Batching / In-flight Batching）**：在token生成过程中动态添加新请求并移除完成请求，最大化GPU吞吐量。

**5) Trade-off（权衡）分析**
- Static Batching实现简单但吞吐低；Continuous Batching吞吐高但调度复杂，需维护每个请求的state（如KV Cache状态）。
- 低TTFT要求高优先级调度，但可能降低整体吞吐量TPS。

**6) 如何确定最优解**
通过压测（Load Testing）对比Static Batching与Continuous Batching在不同负载下的TTFT与TPS。选择Continuous Batching配合请求优先级队列，以满足TTFT < 200ms且TPS >= 5000 tokens/s的目标。

**7) 名词和缩写全称及解释**
- **LLM (Large Language Model)**：大型语言模型。
- **TTFT (Time to First Token)**：从发送请求到接收到第一个生成token的时间。
- **TPOT (Time Per Output Token)**：生成每个输出token所需的平均时间。
- **TP99**：99%的请求延迟小于该值，用于衡量长尾延迟。
- **QPS (Queries Per Second)**：每秒查询率。
- **TPS (Tokens Per Second)**：每秒生成的token数量。
- **HBM (High Bandwidth Memory)**：高带宽内存，GPU常用显存类型。

---