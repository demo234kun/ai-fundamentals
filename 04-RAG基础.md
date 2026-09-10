# 04 RAG 基础（Retrieval-Augmented Generation）

## 第一部分：基本原理

### 0. 为什么需要 RAG

LLM 的三个硬伤：
1. **知识截止**：训练后发生的事不知道；
2. **幻觉**：不知道的事会一本正经地编；
3. **不可溯源**：无法给出信息出处。

RAG 的核心思想（Lewis et al., 2020 提出时即点明）：
> **把生成模型当作黑盒，外挂一个"可更新的非参数化记忆"（检索库）**。参数化记忆（权重）学语言与推理能力，非参数化记忆存事实。知识变了改数据库即可，不用重训模型；检索到的证据随答案一起给出，可溯源。

```
用户问题 q
   │
   ▼
① 检索器 Retriever ──→ 文档库 D（向量库/全文索引/知识图谱）
   │   返回 top-k 相关片段 [d1, d2, ..., dk]
   ▼
② 拼接增强 Prompt：q + 证据 + "仅根据以上资料回答，注明出处"
   │
   ▼
③ 生成器 Generator（LLM）──→ 带引用的回答
```

### 1. 索引（Indexing）：数据入库的预处理流水线

```
原始文档（PDF/Word/网页/数据库）
   → 解析（parsing）：OCR、表格结构还原、版面分析（layout analysis）
   → 分块（chunking）
   → 向量化（embedding）
   → 写入向量库（可附元数据过滤字段）
```

**分块策略**（chunking 是 RAG 质量的第一杠杆）：
| 策略 | 做法 | 适用 |
|---|---|---|
| 固定大小 | 每 512 token、重叠 10-20% |  baseline |
| 递归切分 | 先按段落→句子→词递分割，保持语义单元完整 | 通用首选 |
| 语义切分 | 用 embedding 相似度检测主题漂移点切分 | 讲座/长文 |
| 结构感知 | 按标题层级、表格、代码 AST 切分 | 文档/PDF/代码 |
| 父子块 | 小块用于精确检索，命中后返回其父块供生成 | 长文档 |

**嵌入模型（Embedding）**：把文本映射为稠密向量，语义相近 → 向量空间距离近（余弦相似度）。选型维度：语种、领域、指令前缀、上下文长度、是否归一化。中文场景常用 BGE-M3、bge-large-zh、jina-embeddings-v3；多语言 E5/mE5。

### 2. 检索（Retrieval）

**稠密检索 vs 稀疏检索**：
- **稀疏（BM25/TF-IDF）**：词项匹配，精确、可解释，对专有名词/编号强，但不理解语义（"如何退款"匹配不到"退货政策"）。
- **稠密（向量检索）**：语义匹配，理解 paraphrase，但对精确 token 弱。
- **混合检索（Hybrid）**：两者并行，RRF（Reciprocal Rank Fusion）融合排序，**生产环境默认答案**。

**进阶检索范式**：
- **HyDE**（Hypothetical Document Embedding）：先让 LLM 编一段"假设的理想答案"，用它去检索——问题很短很抽象时效果显著。
- **查询改写 / 多查询扩展**：把口语问题改写成 3-5 个检索友好的子查询并行检索。
- **查询路由（Query Routing）**：分类问题类型，走不同检索器（结构化 SQL / 非结构化向量 / 关键词）。
- **重排序（Rerank）**：双塔召回模型快但粗 → 用交叉编码器（bge-reranker、Cohere Rerank）对 top-100 精排成 top-5，**性价比最高的单点优化**。
- **多向量/ColBERT**：每个 token 一个向量，细粒度 late-interaction，精度高、存储大。

### 3. 生成与引用

- **上下文构造**：相关片段排序（最相关放两端或开头，中间放次相关——"lost in the middle"效应）、去重、token 预算管理。
- **引用溯源**：要求模型输出 `[1][2]` 标注，映射回文档 ID；评估引用支持率。
- **拒答机制**：证据不足时应说"资料中未提及"而非硬编（faithfulness）。

### 4. 高级 RAG 范式

| 范式 | 机制 | 解决什么 |
|---|---|---|
| **RAPTOR** | 递归聚类摘要：叶=原文块，向上聚类生成摘要树，检索可在多抽象层级进行 | 跨章节全局性问题 |
| **GraphRAG** | 抽取实体-关系建知识图谱，社区检测后对每个社区生成摘要；回答全局问题时用"社区摘要的 Map-Reduce" | "整个语料库的主题是什么"这类全局聚合问题 |
| **Self-RAG** | 训练时加入四类反思 token（检索/相关/支持/有用），推理时按需自决定是否检索、自我评判 | 减少过度检索与无用引用 |
| **CRAG** | 检索评估器给检索结果打分（正确/模糊/错误），错误则触发网络搜索兜底 | 低质量知识库下的鲁棒性 |
| **Agentic RAG** | 把检索交给 agent：自主决定检索几次、改写成什么查询、是否需要 rerank/网页搜索 | 复杂多跳问题的动态编排 |

### 5. 结构化数据与 Text-to-SQL

企业 RAG 的另一半是数据库：schema 检索（找相关表/列）→ SQL 生成 → 执行 → 结果让 LLM 用自然语言解读；失败时把报错回灌自纠错。代表：Vanna、DB-GPT、WrenAI。

### 6. 评估（Evaluation）

- **检索侧**：Recall@k、MRR、nDCG、命中率。
- **生成侧**：Faithfulness（答案是否被证据支持）、Answer Relevance、Context Precision/Recall。
- **框架**：RAGAS（自动化指标 + LLM-as-Judge）、TruLens、Arize Phoenix（可观测性）。
- **注意**：端到端评估优先用**真实用户问题集 + 人工抽检**，纯自动指标与体感常不一致。

## 第二部分：主要论文

### 【核心论文】

1. **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks**（Lewis et al., Facebook AI + UCL/NYU, 2020）——RAG 命名与范式确立：生成器联合检索器训练。
2. **Dense Passage Retrieval for Open-Domain QA**（Karpukhin et al., Facebook, 2020）——DPR：双塔稠密检索打败 BM25，向量检索时代开端。
3. **REALM / ORQA**（Guu et al., Google, 2020）——检索与预训练联合优化的另一条路线。
4. **In-Context Retrieval Augmented Language Modeling**（Ram et al., Meta, 2023）——证明检索增强预训练可扩大模型有效参数。
5. **Retrieval-Augmented Generation for Large Language Models: A Survey**（Gao et al., 2023）——Naive/Advanced/Modular RAG 三分法的经典综述。
6. **Lost in the Middle**（Liu et al., 2023）——长上下文中部信息被忽视，指导上下文排列策略。
7. **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection**（Asai et al., 2023）。
8. **Corrective Retrieval Augmented Generation（CRAG）**（Yan et al., 2024）。
9. **From Local to Global: A Graph RAG Approach to Query-Focused Summarization**（Edge et al., Microsoft, 2024）——GraphRAG。
10. **RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval**（Sarthi et al., 2024）。
11. **HyDE: Precise Zero-Shot Dense Retrieval without Relevance Labels**（Gao et al., 2022）。
12. **RAGAS: Automated Evaluation of RAG**（Es et al., 2023）+ **A Survey on Evaluation of LLM-based Agents / RAG evals**（2024-2025）——评估方法学。

### 【应用领域：领域 + 主要创新】

1. **企业知识库问答 + 深度文档理解**：RAGFlow、Dify、AnythingLLM——版面分析（表格/双栏/公式）成为主战场，"垃圾进垃圾出"的解析层决定上限。
2. **法律领域 + 法条级精确引用**：LawGPT-zh、LexGLUE 系列——法条编号必须逐字命中 → 混合检索 + 强制引用校验。
3. **医疗领域 + 证据分级**：MedRAG、MedCite——指南/文献分级证据（RCT > 观察研究）融入重排。
4. **代码检索 + 语义代码搜索**：voyage-code / CodeSearchNet / repo-level RAG（RepoCoder）——按 AST 分块、以调用关系做图检索。
5. **金融投研 + 多源异构融合**：`cn-finance-data`/`ifind` 插件范式——结构化行情 + 非结构化研报 + 实时新闻的混合检索与时效性过滤。
6. **多模态 RAG + 图文联合嵌入**：ColPali（视觉 PDF 嵌入，绕过解析）、MM-RAG——扫描件/图表直读。
7. **对话式 RAG + 多轮指代消解**：Follow-up 问题改写（contextual compression + 指代消解成独立问题）。
8. **推荐/搜索 + 生成式检索（GenIR）**：推荐系统直接"生成"商品 ID（TIGER、LC-Rec），检索与生成边界消融。
9. **长文档理解 + 无限上下文替代方案**：RAPTOR / HippoRAG（记忆式图谱）——百万 token 文档的分层摘要 vs 直接塞长上下文的成本之争。
10. **Agentic RAG + 研究型任务**：Deep Research 类系统（OpenAI / GPT Researcher / STORM）——检索-阅读-引用循环由 agent 自主编排。

## 第三部分：项目学习库（Coding 链接）

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| run-llama/llama_index | RAG 数据框架的事实标准：索引、检索器、查询引擎全抽象 | https://github.com/run-llama/llama_index |
| langchain-ai/langchain（retrieval 模块） | 文档加载器/分块器/向量库接口生态最全 | https://github.com/langchain-ai/langchain |
| infiniflow/ragflow | 深度文档理解的开源 RAG 引擎，OCR 与版面分析教科书 | https://github.com/infiniflow/ragflow |
| langgenius/dify | 低代码 LLM 应用平台：可视化搭 RAG 工作流，看生产化设计 | https://github.com/langgenius/dify |
| deepset-ai/haystack | 工业级 NLP/RAG 流水线，pipeline 抽象清晰 | https://github.com/deepset-ai/haystack |
| microsoft/graphrag | 微软 GraphRAG 官方实现 | https://github.com/microsoft/graphrag |
| HKUDS/LightRAG | 轻量 GraphRAG 实现，代码比原版好读得多 | https://github.com/HKUDS/LightRAG |
| explodinggradients/ragas | RAG 评估框架官方实现 | https://github.com/explodinggradients/ragas |
| facebookresearch/faiss | 向量索引底层库（IVF/HNSW/PQ 全在这），理解检索为什么快 | https://github.com/facebookresearch/faiss |
| qdrant/qdrant / milvus-io/milvus | 生产级向量数据库两强（Qdrant Rust 轻量、Milvus 分布式重） | https://github.com/qdrant/qdrant |
| vanna-ai/vanna | Text-to-SQL 的最佳入门框架 | https://github.com/vanna-ai/vanna |
| jina-ai/jina | 神经搜索框架（含 reranker、embeddings） | https://github.com/jina-ai/jina |
| stanford-oval/storm | 检索+生成写调研报告的 agentic RAG 实例 | https://github.com/stanford-oval/storm |

**动手路线建议**：
1. 用 `llama_index` 20 行代码跑通最小 RAG（加载 PDF → 分块 → 检索 → 回答）。
2. 对比实验：BM25 vs 向量 vs Hybrid+RRF vs 再加 reranker，看同一个问题的 evidence 质量变化。
3. 用 `LightRAG` 在一个小语料上跑 GraphRAG，对比它在"全局性问题"上与普通 RAG 的差距。
4. 用 `ragas` 建 20 条测试集给自己的 RAG 打分，体会自动评估指标与体感的偏差。
