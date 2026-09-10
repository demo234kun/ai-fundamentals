# 04 代码学习库：RAG 基础

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
| facebookresearch/faiss | 向量索引底层库（IVF/HNSW/PQ），理解检索为什么快 | https://github.com/facebookresearch/faiss |
| qdrant/qdrant | 生产级向量数据库（Rust，轻量） | https://github.com/qdrant/qdrant |
| milvus-io/milvus | 生产级向量数据库（分布式） | https://github.com/milvus-io/milvus |
| vanna-ai/vanna | Text-to-SQL 的最佳入门框架 | https://github.com/vanna-ai/vanna |
| jina-ai/jina | 神经搜索框架（含 reranker、embeddings） | https://github.com/jina-ai/jina |
| stanford-oval/storm | 检索+生成写调研报告的 agentic RAG 实例 | https://github.com/stanford-oval/storm |

**动手路线**：
1. 用 `llama_index` 20 行代码跑通最小 RAG（加载 PDF → 分块 → 检索 → 回答）。
2. 对比实验：BM25 vs 向量 vs Hybrid+RRF vs 再加 reranker，看同一问题的 evidence 质量变化。
3. 用 `LightRAG` 在小语料上跑 GraphRAG，对比它在"全局性问题"上与普通 RAG 的差距。
4. 用 `ragas` 建 20 条测试集给自己的 RAG 打分，体会自动指标与体感的偏差。
