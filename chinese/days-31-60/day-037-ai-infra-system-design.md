### Day 37: 推测解码（Speculative Decoding）与Draft-Verify机制

**1) 题目与考察核心**
设计加速LLM生成速度的推理系统。考察核心：Speculative Decoding原理及其对TTFT与TPOT的影响。

**2) 需求澄清与指标定义**
- **QPS**：1000。
- **延迟指标**：目标将TPOT（Time Per Output Token）降低50%。
- **吞吐量TPS**：提升至12000 tokens/s。

**3) 核心架构/技术组件设计**
- Draft Model（小模型）与Verify Model（大模型）双引擎架构。
- Draft Model生成候选tokens，Verify Model进行并行验证（Verify）。

**4) 关键技术深入与可能解**
- **Speculative Decoding**：使用小模型（Draft Model）快速生成多个候选token，然后由大模型（Verify Model）用一次forward验证这些token。若验证通过，则一次性接受多个token，减少自回归步数。
- **Draft-Verify机制**：Draft model生成k个候选，Verify model并行计算损失/概率。接受率（Acceptance Rate）决定加速比。

**5) Trade-off（权衡）分析**
- Speculative Decoding增加系统复杂度（需部署双模型），且Draft Model与Verify Model的架构需匹配（如相同tokenizer）。加速比依赖于acceptance rate。

**6) 如何确定最优解**
选择参数规模为主模型1/4到1/8的Draft Model（如7B draft + 33B verify），通过调优k值（候选数量）使acceptance rate > 60%，从而实现TPOT显著降低。

**7) 名词和缩写全称及解释**
- **Speculative Decoding**：推测解码，通过小模型生成候选token并由大模型验证以加速生成。
- **Draft Model**：用于生成候选token的小参数模型。
- **Verify Model**：用于验证候选token的大参数主模型。
- **Acceptance Rate**：候选token被大模型接受的比率。

---

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 高级推理服务）**：
  ```mermaid
  graph TD
      A[负载均衡器] --> B[推理 Pods vLLM]
      B --> C[PagedAttention KV Cache]
      B --> D[GPU资源管理器]
      D --> E[Kubernetes / Karpenter]
      B --> F[遥测 DCGM/OpenTelemetry]
  ```

- **数据流图（Data Flow Diagram - 连续批处理）**：
  ```mermaid
  flowchart LR
      A[ incoming 请求] --> B[连续批处理调度器]
      B --> C[KV Cache 查找与更新]
      C --> D[GPU Tensor Core 计算]
      D --> E[输出 Tokens]
      E --> F[流式响应]
  ```
