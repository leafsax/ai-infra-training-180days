### Day 112: Model Registry Architecture: Metadata, Versioning, and Artifacts

#### 1) 题目与考察核心
深入设计模型注册表的元数据管理、版本控制和artifact存储架构。

#### 2) 需求澄清与指标定义
- **版本数量**：每个模型平均50个版本。
- **Artifact大小**：单个模型权重文件 10GB-100GB。

#### 3) 核心架构/技术组件设计
- **版本控制策略**：语义化版本（SemVer）或步数版本（Step-based）。
- **Artifact哈希验证**：使用SHA-256确保模型文件完整性。

#### 4) 关键技术深入与可能解（对比分析不同方案）
- **Git-like版本控制 vs 数据库版本控制**：Git适合代码，数据库适合二进制artifacts。

#### 5) Trade-off（权衡）分析
- **灵活性 vs 查询性能**：Git-like灵活但查询慢；数据库查询快但管理二进制文件复杂。

#### 6) 如何确定最优解
采用数据库管理元数据和版本关系，S3存储artifact，通过哈希链接。

#### 7) 名词和缩写全称及解释
- **SemVer (Semantic Versioning)**：语义化版本控制，格式为 Major.Minor.Patch。
- **SHA-256**：安全哈希算法，用于验证数据完整性。

---

