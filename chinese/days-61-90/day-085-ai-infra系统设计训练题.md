### Day 85: 模型服务自动扩缩容 (Model Serving Auto-Scaling)

**1) 题目与考察核心**：
设计 vLLM/TGI 服务的自动扩缩容，考察动态批处理（Dynamic Batching）与模型卸载（Model Unloading）。

**2) 需求澄清与指标定义**：
- 批处理策略：Continuous Batching。
- 模型卸载：空闲 > 10 分钟卸载模型到 CPU 存储。

**3) 核心架构/技术组件设计**：
- 服务引擎：vLLM 或 TGI。
- 扩缩容：基于请求队列长度与 TTFT 指标。

**4) 关键技术深入与可能解**：
- Static Batching vs Continuous Batching：Static 按批次固定大小，Continuous 动态加入新请求。

**5) Trade-off（权衡）分析**：
- Continuous Batching 提高吞吐量但增加调度复杂度。

**6) 如何确定最优解**：
采用 vLLM + Continuous Batching + PagedAttention，结合 KEDA 基于队列长度扩缩容。

**7) 名词和缩写全称及解释**：
- **Static Batching**：静态批处理，将请求按固定批次大小分组。
- **Continuous Batching**：连续批处理（也称 In-flight Batching），在生成过程中动态插入新请求。
- **PagedAttention**：vLLM 使用的内存管理技术，将 KV Cache 分页存储。

---

