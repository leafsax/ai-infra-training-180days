### Day 130: 大规模AI训练的数据预处理与Pipeline
**1) 题目与考察核心**  
设计LLM训练的数据预处理与加载Pipeline，确保GPU不等待数据。

**2) 需求澄清与指标定义**  
- 数据规模：30 TB预处理后数据。
- 吞吐需求：Data Loader需持续提供Batch，GPU利用率 > 90%。
- 延迟：数据Prefetch延迟 < 100 ms。

**3) 核心架构/技术组件设计**  
- 数据Pipeline：原始数据 -> 清洗/Tokenization -> 存储为Binary格式（如WebDataset, TFRecord） -> 异步Data Loader。
- 框架：PyTorch `DataLoader` with `prefetch_factor`, 或 Ray Data, Megatron-LM data pipeline。

**4) 关键技术深入与可能解**  
- **WebDataset vs TFRecord**: WebDataset适合分布式分片读取；TFRecord适合Tensor序列化。
- **Prefetching**: 预取多个Batch，掩盖I/O和Tokenization延迟。

**5) Trade-off（权衡）分析**  
- 预处理在训练前完成（离线） vs 在线预处理：离线预处理存储成本高但训练时无CPU瓶颈；在线预处理节省存储但占用GPU/CPU资源。

**6) 如何确定最优解**  
采用离线Tokenization生成Binary分片文件，训练时使用多进程DataLoader + Prefetching。

**7) 名词和缩写解释**  
- **Tokenization**: 将文本转换为Token（词元）序列的过程。
- **WebDataset**: 用于大规模训练的分布式数据集格式。
- **TFRecord**: TensorFlow的序列化数据格式。
- **DataLoader**: PyTorch中用于加载数据的组件。

---

