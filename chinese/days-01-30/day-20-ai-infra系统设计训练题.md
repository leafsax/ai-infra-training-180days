## 第20天：混合精度训练与梯度检查点

### 1) 题目与考察核心
**题目**：优化LLM训练显存与速度，采用混合精度和激活检查点技术。
**考察核心**：FP16/BF16混合精度，Gradient Checkpointing（激活检查点）。

### 2) 需求澄清与指标定义
- **精度格式**：BF16（Bfloat16）或 FP16。
- **指标**：显存减少50%，训练速度提升1.5-2倍。

### 3) 核心架构/技术组件设计
- **Automatic Mixed Precision (AMP)**：PyTorch的`torch.cuda.amp`，自动选择操作符的精度。
- **Activation Checkpointing**：不保存所有中间激活值，而是在Backward时重新计算（Recompute）。

### 4) 关键技术深入与可能解
- *BF16 vs FP16*：BF16范围与FP32相同，避免溢出；FP16精度更高但易溢出。LLM训练首选BF16。
- *Gradient Checkpointing*：只保存部分层的激活，Backward时重新计算其余层，显存节省可达50%，计算增加约33%。

### 5) Trade-off（权衡）分析
- *Checkpting vs 显存*：启用Activation Checkpointing大幅减少显存，但增加Forward/Backward的计算时间（Trade-off：显存换计算）。

### 6) 如何确定最优解
- 默认启用BF16 AMP和Activation Checkpointing（`gradient_checkpointing=True`），在显存允许的情况下减少Checkpting粒度以提速。

### 7) 名词和缩写全称及解释
- **FP16 (Float16)**：16位浮点数格式。
- **BF16 (Bfloat16)**：Google提出的16位浮点格式，保持与FP32相同的指数范围。
- **AMP (Automatic Mixed Precision)**：自动混合精度训练。
- **Gradient/Activation Checkpointing**：梯度/激活检查点，通过重新计算激活值来节省显存。

---