## 第21天：推理与训练量化（INT8, INT4, AWQ, GPTQ, FP8）

### 1) 题目与考察核心
**题目**：设计模型量化pipeline，将70B模型量化为INT4/FP8以部署到推理集群。
**考察核心**：PTQ（训练后量化），AWQ，GPTQ，FP8训练与推理。

### 2) 需求澄清与指标定义
- **目标精度**：INT4或FP8。
- **指标**：量化后模型PPL（困惑度）增加 < 1%，显存减少50%-75%，吞吐提升2x。

### 3) 核心架构/技术组件设计
- **Quantization Engine**：使用llama.cpp, AutoAWQ, 或 Hugging Face `transformers` quantization API。
- **Calibration Dataset**：用于调整量化参数的数百条样本数据。

### 4) 关键技术深入与可能解
- *AWQ (Activation-aware Weight Quantization)*：根据激活值分布保护重要权重，INT4量化效果好。
- *GPTQ*：基于二阶导数的量化算法，逐层量化权重，精度高但速度慢。
- *FP8 (Float8)*：NVIDIA H100支持的8位浮点格式，训练和推理均支持，性能提升显著。

### 5) Trade-off（权衡）分析
- *INT4 vs FP8*：INT4显存节省最多但需专用硬件内核（如Tensor Core FP8或INT4）；FP8在H100上原生支持，训练和推理均可用，精度损失小。

### 6) 如何确定最优解
- 推理部署优先使用AWQ INT4或GPTQ INT4（兼容广泛硬件）；H100集群训练/推理优先使用FP8混合精度。

### 7) 名词和缩写全称及解释
- **PTQ (Post-Training Quantization)**：训练后量化。
- **AWQ (Activation-aware Weight Quantization)**：感知激活的权重量化。
- **GPTQ (Generative Pre-trained Transformer Quantization)**：基于二阶信息的量化算法。
- **FP8 (Float8)**：8位浮点数格式，支持更高训练/推理吞吐。
- **PPL (Perplexity)**：困惑度，衡量语言模型预测质量的指标。

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
