### Day 36: GPU显存优化：模型量化技术（INT8, INT4, FP8, AWQ, GPTQ）

**1) 题目与考察核心**
设计支持大模型显存优化的推理服务。考察核心：模型量化技术及其对精度与性能的影响。

**2) 需求澄清与指标定义**
- **模型规模**：33B参数模型。
- **显存HBM**：4x A100 80GB（需通过量化将模型从FP16的66GB压缩至INT4的18GB左右）。
- **延迟指标**：TTFT < 300ms，TP99 < 4秒。

**3) 核心架构/技术组件设计**
- 量化引擎：在模型部署前或部署时进行权重量化（Weight Quantization）或激活量化（Activation Quantization）。
- 推理引擎：支持INT4/INT8/FP8张量计算（如NVIDIA Tensor Cores支持FP8）。

**4) 关键技术深入与可能解**
- **FP16/BF16**：原生半精度，显存占用大但精度无损。
- **INT8量化**：权重与激活量化为8位，显存减半，精度损失较小。
- **INT4量化（AWQ/GPTQ）**：AWQ（Activation-aware Weight Quantization）与GPTQ（Generative Pre-trained Transformer Quantization）按权重重要性进行4位量化，显存占用极低。
- **FP8**：NVIDIA Hopper架构支持的原生8位浮点，兼顾性能与精度。

**5) Trade-off（权衡）分析**
- 量化程度越高，显存节省越多，但精度下降风险增加。INT4可能引起特定领域任务性能下降。

**6) 如何确定最优解**
通过Validation Set评估FP8、INT8、INT4(GPTQ)的准确率下降。若下降<1%，选择INT4/GPTQ以最大化显存利用率与吞吐量。

**7) 名词和缩写全称及解释**
- **量化（Quantization）**：将模型权重/激活的浮点数精度降低（如FP16->INT8）以减少显存与计算开销。
- **AWQ (Activation-aware Weight Quantization)**：考虑激活值分布的权重量化方法。
- **GPTQ (Generative Pre-trained Transformer Quantization)**：基于一次校准的量化算法，支持INT4精度。
- **FP8**：8位浮点数格式，NVIDIA Hopper架构原生支持。

---