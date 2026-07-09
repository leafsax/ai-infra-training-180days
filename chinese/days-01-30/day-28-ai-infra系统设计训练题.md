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

### **8) 组件图与数据流图**

- **组件图（Component Diagram - 分布式训练）**：
  ```mermaid
  graph TD
      A[数据加载器] --> B[训练节点 GPU+CPU]
      B --> C[NCCL All-Reduce 网络]
      B --> D[检查点存储 S3/NFS]
      B --> E[模型注册表 MLflow]
      C --> B
  ```

- **数据流图（Data Flow Diagram - 训练流程）**：
  ```mermaid
  flowchart LR
      A[训练数据] --> B[各GPU梯度计算]
      B --> C[NCCL梯度同步]
      C --> D[权重更新]
      D --> E[检查点保存]
  ```


### **9) 面试官追问环节（尖锐且挑剔）**

- **关于推理批处理与延迟**：你提到了静态批处理与连续批处理。在中途动态拆分批处理时，内核融合（kernel fusion）的确切开销是多少？当长上下文请求（如128K token）到达并迫使为较短请求进行KV cache驱逐时，如何保证TP99延迟？
- **关于分布式训练通信**：在DP/TP/PP中，如何在1024个GPU的集群中处理straggler（慢节点）节点？如果一个GPU慢5%，整个All-Reduce步骤会阻塞吗？对于使用BF16权重和梯度的NCCL All-Reduce，每步的确切通信量是多少？
- **关于内存优化**：你提到了ZeRO和PagedAttention。对于ZeRO-3，如何在优化器步骤期间最小化跨节点通信开销？对于PagedAttention，当HBM发生碎片化时，确切的驱逐策略是什么（LRU vs LFU vs 基于注意力分数）？
