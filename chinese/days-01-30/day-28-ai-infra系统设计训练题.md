## 第28天：边缘AI与设备端LLM推理（Edge AI & On-Device Inference）

### 1) 题目与考察核心
**题目**：设计在边缘设备（如手机、边缘服务器）上运行7B-14B LLM的推理系统。
**考察核心**：ONNX, TensorRT-LLM for Edge, NPU/QNN优化，端侧量化。

### 2) 需求澄清与指标定义
- **设备约束**：端侧GPU/NPU显存/内存 8GB-16GB。
- **指标**：端侧生成速度 > 20 Tokens/s，功耗 < 15W。

### 3) 核心架构/技术组件设计
- **Model Conversion Pipeline**：PyTorch -> ONNX -> TensorRT / QNN。
- **Quantization**：INT4/INT8量化适配NPU硬件。

### 4) 关键技术深入与可能解
- *TensorRT-LLM*：NVIDIA的推理优化库，支持将LLM编译为高效GPU内核。
- *QNN (Qualcomm AI Engine Direct)*：高通NPU的推理引擎，支持端侧LLM部署。

### 5) Trade-off（权衡）分析
- *云端 vs 端侧*：云端算力无限但延迟高、隐私风险；端侧延迟低、隐私好但算力和显存受限，需重度量化。

### 6) 如何确定最优解
- 对于7B-14B模型，采用INT4量化并使用TensorRT-LLM或MLC LLM部署到支持CUDA/NPU的边缘设备。

### 7) 名词和缩写全称及解释
- **ONNX (Open Neural Network Exchange)**：开放的机器学习模型交换格式。
- **TensorRT-LLM**：NVIDIA优化的LLM推理库，基于TensorRT。
- **NPU (Neural Processing Unit)**：神经网络处理单元，专为AI计算优化的硬件。

---