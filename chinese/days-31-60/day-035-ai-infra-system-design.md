### Day 35: 推理服务框架对比：vLLM vs TGI vs TensorRT-LLM

**1) 题目与考察核心**
为高吞吐LLM推理服务选择合适的框架。考察核心：主流推理框架的架构特点与适用场景。

**2) 需求澄清与指标定义**
- **QPS**：3000。
- **延迟指标**：TTFT < 200ms，TP99 < 3秒。
- **吞吐量TPS**：10000 tokens/s。

**3) 核心架构/技术组件设计**
- 框架选型层：评估vLLM（基于PagedAttention与Continuous Batching）、TGI（Text Generation Inference，HuggingFace生态）、TensorRT-LLM（NVIDIA底层优化框架）。

**4) 关键技术深入与可能解**
- **vLLM**：开源框架，支持Continuous Batching与PagedAttention，易于集成与定制。
- **TGI**：支持动态批处理、tensor parallelism，与HuggingFace模型无缝对接，支持gRPC/HTTP。
- **TensorRT-LLM**：NVIDIA官方框架，通过算子融合、CUDA Graphs实现极致性能，但需编译生成engine，灵活性较低。

**5) Trade-off（权衡）分析**
- vLLM/TGI开发友好、部署简单；TensorRT-LLM性能最高但适配周期长、需特定硬件（NVIDIA GPU）与编译流程。

**6) 如何确定最优解**
若追求极致吞吐且硬件固定为NVIDIA GPU集群，选择TensorRT-LLM；若需快速迭代与灵活批处理策略，选择vLLM。

**7) 名词和缩写全称及解释**
- **TensorRT-LLM**：NVIDIA推出的针对LLM推理优化的GPU加速库。
- **CUDA Graphs**：NVIDIA CUDA编程模型中的图执行优化技术，减少CPU-GPU调度开销。

---