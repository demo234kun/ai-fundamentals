# AI 基础知识全景知识库

AI 基础知识全景知识库：每个维度下分 **基础知识 / 论文 / 代码** 三个模块。论文 PDF 已全部下载到本地（点击即读），代码模块为精选学习仓库链接。

> 版本：2026-09 整理。论文年份与链接基于整理时的知识库，引用前建议在 arXiv 再核对。

**主线**：预训练模型 →（微调/RL 对齐出好用的模型）→（RAG 给模型外挂知识）→（Agent/Skill 让模型会做事）→（Harness 是承载这一切的运行时）。

## 学习路径

1. **第 1 周**：维度 01 + 04（模型怎么对齐、知识怎么外挂）——QLoRA 微调一个 7B 模型 + 用 llama_index 搭最小 RAG。
2. **第 2 周**：维度 02 + 03（Agent 怎么规划、技能怎么沉淀）——用 smolagents 复现 ReAct，把做过的任务沉淀成 SKILL.md。
3. **第 3 周**：维度 05 + 回读全部论文（harness 决定 Agent 的能力上限，最后再回头看会有顿悟）。

---

## 01 微调与强化学习（LoRA / SFT / DPO / KTO / GRPO）

| 模块 | 内容 | 链接 |
|---|---|---|
| 📘 基础知识 | 微调/RL 全景管线：SFT → LoRA/QLoRA → RLHF+PPO → DPO/KTO → GRPO/RLVR，含公式直觉与工程细节 | [01-微调与强化学习/README.md](./01-微调与强化学习/README.md) |
| 📄 论文（10 篇已下载） | LoRA、QLoRA、InstructGPT、DPO、KTO、DeepSeekMath、DeepSeek-R1、LIMA、RLHF 工程、奖励过优化 | [01-微调与强化学习/papers/](./01-微调与强化学习/papers/README.md) |
| 💻 代码 | LLaMA-Factory、peft、trl、unsloth、verl、OpenRLHF、TinyZero 等 12 个仓库 | [01-微调与强化学习/code.md](./01-微调与强化学习/code.md) |

## 02 Agent 基础（ReAct / Planning / Tool Use / Multi-Agent）

| 模块 | 内容 | 链接 |
|---|---|---|
| 📘 基础知识 | ReAct 循环、Planning（Plan-and-Execute/ToT/Reflexion）、Function Calling → CodeAct → MCP、四层记忆、多 Agent 模式 | [02-Agent基础/README.md](./02-Agent基础/README.md) |
| 📄 论文（10 篇已下载） | ReAct、Toolformer、CoT、ToT、Reflexion、Plan-and-Solve、Generative Agents、Voyager、MemGPT、Agent 综述 | [02-Agent基础/papers/](./02-Agent基础/papers/README.md) |
| 💻 代码 | smolagents、langgraph、AutoGen、crewAI、OpenHands、Agentless、browser-use 等 14 个仓库 | [02-Agent基础/code.md](./02-Agent基础/code.md) |

## 03 Skill 基础（SKILL.md / 程序性记忆）

| 模块 | 内容 | 链接 |
|---|---|---|
| 📘 基础知识 | Skill 与 Tool/RAG/MCP 的边界、渐进式披露三层结构、Voyager 自动沉淀循环、SKILL.md 工程规范 | [03-Skill基础/README.md](./03-Skill基础/README.md) |
| 📄 论文（6 篇已下载） | Voyager、ExpeL、CaMeL、Generative Agents、Reflexion、Agent 综述（+ Agent Skills 白皮书等在线资源） | [03-Skill基础/papers/](./03-Skill基础/papers/README.md) |
| 💻 代码 | anthropics/skills、Voyager、anthropics/courses、MCP servers 等 7 个仓库 | [03-Skill基础/code.md](./03-Skill基础/code.md) |

## 04 RAG 基础（检索增强生成）

| 模块 | 内容 | 链接 |
|---|---|---|
| 📘 基础知识 | 索引/分块、稠密 vs 稀疏 vs 混合检索、HyDE/Rerank、RAPTOR/GraphRAG/Self-RAG/CRAG/Agentic RAG、RAGAS 评估 | [04-RAG基础/README.md](./04-RAG基础/README.md) |
| 📄 论文（12 篇已下载） | RAG、DPR、REALM、In-Context RALM、RAG 综述、Lost in the Middle、Self-RAG、CRAG、GraphRAG、RAPTOR、HyDE、RAGAS | [04-RAG基础/papers/](./04-RAG基础/papers/README.md) |
| 💻 代码 | llama_index、ragflow、dify、LightRAG、graphrag、faiss、qdrant、vanna、storm 等 14 个仓库 | [04-RAG基础/code.md](./04-RAG基础/code.md) |

## 05 Harness 基础（Agent 运行时）

| 模块 | 内容 | 链接 |
|---|---|---|
| 📘 基础知识 | 五大组件：上下文组装 / 工具运行时 / Agent 循环 / 状态记忆 / 交付安全，附 harness 评价清单 | [05-Harness基础/README.md](./05-Harness基础/README.md) |
| 📄 论文（5 篇已下载） | SWE-agent（ACI）、SWE-bench、OpenHands、BrowserGym、OSWorld（+ Aider/Building Effective Agents 等在线资源） | [05-Harness基础/papers/](./05-Harness基础/papers/README.md) |
| 💻 代码 | OpenHands、SWE-agent、aider、open-interpreter、MCP SDK、terminal-bench、ART 等 11 个仓库 | [05-Harness基础/code.md](./05-Harness基础/code.md) |

---

## 统计

| 维度 | 论文 PDF | 代码仓库 |
|---|---|---|
| 01 微调与强化学习 | 10 | 12 |
| 02 Agent 基础 | 10 | 14 |
| 03 Skill 基础 | 6 | 7 |
| 04 RAG 基础 | 12 | 14 |
| 05 Harness 基础 | 5 | 11 |
| **合计** | **43**（含 4 篇跨模块重复存放） | **58** |
