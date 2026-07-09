### Day 108: Multimodal Pretraining vs SFT vs RLHF Infrastructure

#### 1) 题目与考察核心
设计多模态模型的不同训练阶段（Pretraining, SFT, RLHF）的基础设施。

#### 2) 需求澄清与指标定义
- **Pretraining数据**：100亿图像-文本对。
- **SFT数据**：100万指令对。
- **RLHF数据**：10万偏好对。
- **SFT训练时间目标**：≤ 2周。

#### 3) 核心架构/技术组件设计
- **Pretraining集群**：大规模无监督训练，强调吞吐量。
- **SFT集群**：监督微调，强调数据多样性。
- **RLHF集群**：包含Reward Model训练和PPO强化学习，计算密集。

#### 4) 关键技术深入与可能解（对比分析不同方案）
- **统一集群 vs 专用集群**：
  - *统一集群*：灵活但资源竞争。
  - *专用集群*：性能稳定，但成本高。

#### 5) Trade-off（权衡）分析
- **成本 vs 性能隔离**：专用集群避免干扰但增加资本支出。

#### 6) 如何确定最优解
采用阶段隔离：Pretraining使用大规模DP集群，SFT/RLHF使用混合并行集群。

#### 7) 名词和缩写全称及解释
- **Pretraining（预训练）**：在大规模无标注数据上训练基础模型。
- **SFT (Supervised Fine-Tuning)**：监督微调，使用指令-回答对训练。
- **RLHF (Reinforcement Learning from Human Feedback)**：基于人类反馈的强化学习。

---

