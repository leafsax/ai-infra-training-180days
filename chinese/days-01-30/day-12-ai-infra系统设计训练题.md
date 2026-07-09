## 第12天：GPU显存卸载（CPU RAM / NVMe Offloading）

### 1) 题目与考察核心
**题目**：当GPU显存不足以加载大模型时，设计显存卸载架构以支持推理。
**考察核心**：CPU Offloading, NVMe Offloading, 动态卸载策略。

### 2) 需求澄清与指标定义
- **模型大小**：70B FP16 = 140GB，单卡80GB GPU。
- **指标**：启用Offloading后，TTFT增加不超过3倍，吞吐下降不超过50%。

### 3) 核心架构/技术组件设计
- **Offload Engine**：在计算前将权重从CPU RAM或NVMe加载到GPU HBM，计算后卸载回。
- **Layer-wise Offloading**：按Layer逐步卸载和加载。

### 4) 关键技术深入与可能解
- *CPU Offloading*：通过PCIe传输，带宽~32 GB/s，延迟高。
- *NVMe Offloading*：使用高速NVMe SSD（如PCIe 4.0 x4, ~7 GB/s），比CPU RAM更慢，但容量大。

### 5) Trade-off（权衡）分析
- *Offloading vs OOM*：Offloading允许运行超大模型，但PCIe/NVMe带宽成为瓶颈，导致TTFT和每Token延迟显著增加。

### 6) 如何确定最优解
- 对于推理，优先使用模型量化（INT4/FP8）或Tensor Parallelism跨多卡。仅在无足够GPU显存时启用Offloading（如vLLM的`--offload-size`）。

### 7) 名词和缩写全称及解释
- **Offloading (卸载)**：将数据或计算从GPU显存转移到CPU内存或磁盘，以节省显存。
- **NVMe (Non-Volatile Memory express)**：非易失性内存主机控制器接口，高速SSD标准。

---