### Day 47: 异步推理与流式响应（Streaming Responses）设计

**1) 题目与考察核心**
设计支持流式输出的LLM推理API。考察核心：异步处理、SSE（Server-Sent Events）与流式生成。

**2) 需求澄清与指标定义**
- **QPS**：2000。
- **延迟指标**：TTFT < 200ms，流式token间隔（Inter-token latency） < 50ms。

**3) 核心架构/技术组件设计**
- API层：支持SSE或gRPC流式传输。
- 推理引擎：支持逐token输出（Streaming Output）而非等待全部生成完成。

**4) 关键技术深入与可能解**
- **SSE (Server-Sent Events)**：单向HTTP流，适合推送token序列。
- **gRPC Streaming**：双向或服务端流，延迟更低但客户端实现复杂。

**5) Trade-off（权衡）分析**
- SSE兼容性好但传输效率略低；gRPC流式性能高但需客户端支持gRPC。

**6) 如何确定最优解**
对外部API采用SSE以兼容广泛客户端；内部微服务通信采用gRPC Streaming以降低延迟。

**7) 名词和缩写全称及解释**
- **SSE (Server-Sent Events)**：服务器向客户端推送事件的HTTP标准。
- **Inter-token latency**：生成连续两个token之间的时间间隔。

---