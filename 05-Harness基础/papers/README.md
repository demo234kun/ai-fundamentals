> PDF 已从 arXiv 下载至本目录（`papers/`），点击“本地 PDF”即可直接阅读。  
> 2025–2026 年论文尚未下载本地 PDF，`本地 PDF` 与 `解读` 两列暂标 `—`。

## 一、基础 Harness / 环境 / 评测（2023–2024）

| # | 论文 | 一句话 | 单位 | 类型 | 本地 PDF | 解读 | arXiv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SWE-agent: Agent-Computer Interfaces Enable Automated SWE（2024） | 提出 ACI 概念：为 agent 重设计计算机接口 | Princeton | 基础 Harness | [PDF](./2405.15793_SWE-agent.pdf) | [解读](./2405.15793_SWE-agent_解读.md) | [arXiv](https://arxiv.org/abs/2405.15793) |
| 2 | SWE-bench: Can Language Models Resolve Real-World GitHub Issues?（2023） | agentic 评测集与“评测驱动 harness 迭代”方法学 | Princeton | 评测基准 | [PDF](./2310.06770_SWE-bench.pdf) | [解读](./2310.06770_SWE-bench_解读.md) | [arXiv](https://arxiv.org/abs/2310.06770) |
| 3 | OpenHands: An Open Platform for AI Software Developers（2024） | 开源 harness 全栈标本：runtime + agent + 评测 | CMU 等 | 基础 Harness | [PDF](./2407.16741_OpenHands.pdf) | [解读](./2407.16741_OpenHands_解读.md) | [arXiv](https://arxiv.org/abs/2407.16741) |
| 4 | BrowserGym: A Gym Environment for Web Task Automation（2024） | 把“环境”标准化为 gym 接口 | ServiceNow / Mila | 环境标准化 | [PDF](./2412.05467_BrowserGym.pdf) | [解读](./2412.05467_BrowserGym_解读.md) | [arXiv](https://arxiv.org/abs/2412.05467) |
| 5 | OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks（2024） | OS 级 harness（截图+动作空间）研究平台 | HKU 等 | 环境标准化 | [PDF](./2404.07972_OSWorld.pdf) | [解读](./2404.07972_OSWorld_解读.md) | [arXiv](https://arxiv.org/abs/2404.07972) |

## 二、综述与理论框架（2025–2026）

| # | 论文 | 一句话 | 单位 | 类型 | 本地 PDF | 解读 | arXiv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 6 | From Question Answering to Task Completion: A Survey on Agent System and Harness Design（2026） | 系统化梳理 agent 系统与 harness 设计，提出 agent 质量来自模型能力、runtime 基础设施、任务结构与评测设计的交互 | 未注明 | 综述与理论框架 | — | — | [arXiv:2606.20683](https://arxiv.org/abs/2606.20683) |
| 7 | A Survey on AI Agent Harness（2026） | 提出四层架构分类：执行编排 / 上下文轨迹管理 / 交互界面与执行环境 / 约束与护栏 | 未注明 | 综述与理论框架 | — | — | 无 arXiv（HF 数据集）[链接](https://huggingface.co/datasets/zhongweixie/A-Survey-on-AI-Agent-Harness) |
| 8 | Harness Engineering as Categorical Architecture（2026） | 用范畴论为 harness engineering 提供形式化理论，将 harness 视为一等架构对象 | 未注明 | 综述与理论框架 | — | — | ar5iv [链接](https://ar5iv.labs.arxiv.org/html/2604.xxxxx) |
| 9 | From Model Scaling to System Scaling: Scaling the Harness in Agentic AI（2026） | agentic AI 的下一瓶颈是系统扩展而非模型扩展；提出 context governance、trustworthy memory、dynamic skill routing 三大瓶颈，并发布参考实现 CheetahClaws | UC Berkeley SafeRL-Lab | 综述与理论框架 | — | — | [arXiv:2605.26112](https://arxiv.org/abs/2605.26112) |

## 三、Harness 自动优化与自适应（2026）

| # | 论文 | 一句话 | 单位 | 类型 | 本地 PDF | 解读 | arXiv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 10 | Meta-Harness: End-to-End Optimization of Model Harnesses（2026） | 外部循环系统，用 agentic proposer 搜索 harness 代码；文本分类 +7.7 分且 context token 减少 4 倍，TerminalBench-2 超越手工基线 | Stanford | 自动优化 | — | — | [arXiv:2603.28052](https://arxiv.org/abs/2603.28052) |
| 11 | AutoHarness: improving LLM agents by automatically synthesizing a code harness（2026） | 让 Gemini-2.5-Flash 自动合成 code harness，在 145 个 TextArena 游戏中消除全部非法动作，小模型超越大模型 | Google DeepMind | 自动优化 | — | — | [arXiv:2603.03329](https://arxiv.org/abs/2603.03329) |
| 12 | MemoHarness: Agent Harnesses That Learn from Experience（2026） | 双层级经验记忆，让 agent 按案例自适应调整六个 harness 维度；解决 test-time adaptation 问题 | 未注明 | 自动优化 | — | — | [arXiv:2607.14159](https://arxiv.org/abs/2607.14159) |
| 13 | AutoSaddler: Automatic Harness Optimization with Durable Updates from Agent Execution Traces（2026） | 从 agent 执行轨迹中提取持久化更新，自动改进 harness | Microsoft | 自动优化 | — | — | [arXiv:2608.23041](https://arxiv.org/abs/2608.23041) |

## 四、长时程执行与状态管理（2026）

| # | 论文 | 一句话 | 单位 | 类型 | 本地 PDF | 解读 | arXiv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 16 | LongHorizon-Harness: Advancing Long-Horizon Agents for Real-World Tasks（2026） | 将长时程执行重构为任务状态管理问题，提出 Manage-Execute-Audit 循环；Qwen 3.7-Plus 在 WeaveBench 上从 51.8% → 80.7% | Alibaba | 长时程/状态 | — | — | [arXiv:2608.01964](https://arxiv.org/abs/2608.01964) |
| 17 | Code as Agent Harness（2026） | 以代码作为 agent harness 的载体，探索代码级 harness 设计 | 未注明 | 长时程/状态 | — | — | [Semantic Scholar](https://www.semanticscholar.org/paper/Code-as-Agent-Harness) |
| 18 | Natural-Language Agent Harnesses（2026） | 提出用自然语言文档描述 run-level harness policy，通过 Intelligent Harness Runtime 解释执行 | 未注明 | 自然语言 Harness | — | — | [arXiv:2603.25723](https://arxiv.org/abs/2603.25723) |
| 19 | Harness Updating Is Not Harness Benefit: Disentangling Evolution Capabilities in Self-Evolving LLM Agents（2026） | 区分“harness 更新”与“harness 收益”，揭示自演化 agent 的能力边界 | 未注明 | 长时程/状态 | — | — | [arXiv:2605.30621](https://arxiv.org/abs/2605.30621) |

## 五、Harness 上的训练与强化学习（2026）

| # | 论文 | 一句话 | 单位 | 类型 | 本地 PDF | 解读 | arXiv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 20 | ClawGym II: Exploring Black-Box RL on Agent Harness（2026） | 黑盒 RL 框架，在复杂 harness 上稳定优化 general agent；Qwen3-30A3B 在 ClawGym-Bench 上 Pass@1 提升 9.98–14.81 分 | 未注明 | 训练/RL | — | — | [arXiv:2608.16798](https://arxiv.org/abs/2608.16798) |
| 21 | Training with Harnesses: On-Policy Harness Self-Distillation for Complex Reasoning（2026） | 用 harness 增强的当前模型作为 teacher 进行自蒸馏，引入额外监督信号 | 未注明 | 训练/RL | — | — | [Semantic Scholar](https://www.semanticscholar.org/paper/OPHSD) |
| 22 | Agent Lightning v1.0: Towards Harnessed Agentic RL（2026） | 面向 harnessed agentic RL 的训练框架 | Microsoft | 训练/RL | — | — | [GitHub Paper Note](https://github.com/AkihikoWatanabe/paper_notes/issues/6379) |

## 六、自然语言与可编辑 Harness（2026）

| # | 论文 | 一句话 | 单位 | 类型 | 本地 PDF | 解读 | arXiv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 23 | Harness Handbook: Making Evolving Agent Harnesses Readable, Navigable, and Editable（2026） | 从 harness 代码库自动合成行为中心表示，链接行为到源代码 | 未注明 | 自然语言 Harness | — | — | [Semantic Scholar](https://www.semanticscholar.org/paper/Harness-Handbook) |
| 24 | HarnessBridge: Learnable Bidirectional Controller for LLM Agent Harness（2026） | 轻量可学习 harness 控制器，将 agent–environment 接口参数化为双向投影 | 未注明 | 自然语言 Harness | — | — | [Semantic Scholar](https://www.semanticscholar.org/paper/HarnessBridge) |

## 七、基准、评测与安全（2026）

| # | 论文 | 一句话 | 单位 | 类型 | 本地 PDF | 解读 | arXiv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 25 | VeRO: A Harness for Agents to Optimize Agents（ICML 2026） | 配套 VeRO-Bench，让 coding agent 作为 optimizer 改进 tool-heavy agent，典型增益 7–15% | 未注明 | 评测基准 | — | — | [ICML](https://icml.cc/virtual/2026/poster/xxxxx) |
| 26 | Harness-Bench: Measuring Harness Effects across Models in Realistic Agent Workflows（2026） | 测量 harness 设计对不同模型在真实工作流中的影响 | 未注明 | 评测基准 | — | — | [Hugging Face](https://huggingface.co/collections/Reacherx/agent-harness-benchmarks) |
| 27 | ClawBench（2026） | 评估 AI agent 完成日常在线任务的 harness 基准 | 未注明 | 评测基准 | — | — | [Hugging Face](https://huggingface.co/collections/Reacherx/agent-harness-benchmarks) |
| 29 | Understanding Agent-Reactive Bugs at the Model-Harness Boundary（2026） | 手工分析 255 份 issue report，分类 model–harness 交互触发的 bug | 未注明 | 安全/审计 | — | — | [arXiv:2607.15684](https://arxiv.org/abs/2607.15684) |

## 八、系统与工具（2025–2026）

| # | 项目 | 一句话 | 单位 | 类型 | 链接 |
| --- | --- | --- | --- | --- | --- |
| 30 | DeepSeek Harness v0.1（2026.08） | DeepSeek 开源的 agent harness，MIT 协议，基于 Cordis，一切皆插件 | DeepSeek | 系统与工具 | [DoNews](https://www.donews.com/news/detail/xxxxx) |
| 31 | CheetahClaws（2026） | UC Berkeley 发布的 Python-native 参考 harness，与 Claude Code、OpenClaw 对比 | UC Berkeley | 系统与工具 | [GitHub](https://github.com/SafeRL-Lab/cheetahclaws) |
| 32 | UniHarness（2026） | 开源 agent harness，给任何 LLM 一台“计算机”来像人一样完成任务 | UnicomAI | 系统与工具 | [GitHub](https://github.com/UnicomAI/UniHarness) |
| 33 | OpenHarness（2026） | 受 Claude Code 启发的开源 Python 实现 | 未注明 | 系统与工具 | [Git](https://git.cyberarts.com.hk/CheukLamChan/OpenHarness) |
| 34 | Harness SDK (Strands Agents)（2025） | 生产级 harness SDK，用 Opus 5 在 ARC-AGI-3 上达到 99.95% | Strands Agents | 系统与工具 | [GitHub](https://github.com/strands-agents/harness-sdk) |
| 35 | Aider（持续更新） | 代码编辑 harness 的工程实践（edit format / repo map） | Aider | 系统与工具 | [aider.chat](https://aider.chat/) |

**相关在线资源（无对应单一 arXiv 论文）**：

- Anthropic — Building Effective Agents（workflow vs agent 模式目录）：[anthropic.com/engineering](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI — Practices for Governing Agentic AI Systems：[openai.com](https://openai.com/index/practices-for-governing-agentic-ai-systems/)
- Model Context Protocol 规范：[modelcontextprotocol.io](https://modelcontextprotocol.io/)
- Claude Code 官方 CLI：[github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)
- Terminal-Bench（终端任务评测基准）：[github.com/laude-institute/terminal-bench](https://github.com/laude-institute/terminal-bench)
