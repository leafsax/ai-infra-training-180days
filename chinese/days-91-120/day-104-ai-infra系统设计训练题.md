### Day 104: Multimodal Batching: Handling Variable-Length Visual Tokens

#### 1) 题目与考察核心
设计多模态batching策略，处理图像和文本的变长tokens（Variable-Length Visual Tokens）。

#### 2) 需求澄清与指标定义
- **文本token长度**：平均512，最大2048。
- **视觉token长度**：图像4096，视频可达100,000+。
- **GPU显存限制**：80GB HBM，需最大化batch size。

#### 3) 核心架构/技术组件设计
- **Bucket Batching**：将相似长度的样本分到同一batch。
- **Packing/Padding**：对变长序列进行padding或packing以形成矩形batch。

#### 4) 关键技术深入与可能解（对比分析不同方案）
- **Padding vs Packing**：
  - *Padding*：填充到最大长度，简单但浪费计算和显存。
  - *Packing*：将多个短序列拼接，减少padding，但增加索引管理复杂性。

#### 5) Trade-off（权衡）分析
- **显存效率 vs 实现复杂度**：Packing提高利用率但增加数据加载逻辑。

#### 6) 如何确定最优解
采用Bucket Batching + 适度Padding，确保batch内视觉token长度方差最小化。

#### 7) 名词和缩写全称及解释
- **Bucket Batching**：将长度相似的样本分组到同一batch。
- **Padding**：用特殊token填充序列至固定长度。

---

