### Day 38: 推理集群的请求路由与负载均衡策略

**1) 题目与考察核心**
设计多节点推理集群的请求路由与负载均衡系统。考察核心：路由策略、状态感知负载均衡。

**2) 需求澄清与指标定义**
- **集群规模**：20个GPU节点，每节点8x GPU。
- **QPS**：5000。
- **延迟指标**：TP99 < 3秒，要求请求分布均匀，避免热点节点。

**3) 核心架构/技术组件设计**
- 路由层：API Gateway + 服务发现（Service Discovery）+ 状态感知负载均衡器（State-aware Load Balancer）。
- 监控组件：收集各节点的GPU利用率、KV Cache剩余显存、请求队列长度。

**4) 关键技术深入与可能解**
- **Round-Robin / Random Routing**：简单但无视节点状态，易导致负载不均。
- **Least Connections / Least Load**：基于活跃请求数或GPU利用率路由。
- **KV Cache-aware Routing**：将具有相似prompt的请求路由到同一节点，利用KV Cache缓存提升命中率。

**5) Trade-off（权衡）分析**
- 状态感知路由增加路由层复杂度与监控开销；简单路由易实现但吞吐与延迟表现不稳定。

**6) 如何确定最优解**
采用基于GPU利用率与请求队列长度的Least Load路由，结合KV Cache亲和性（Affinity）路由，平衡负载与缓存命中率。

**7) 名词和缩写全称及解释**
- **负载均衡（Load Balancing）**：将网络请求分发到多个服务器以提高响应速度与可靠性。
- **服务发现（Service Discovery）**：动态识别集群中可用服务实例的技术。

---