# 05 论文索引：Harness 基础

> PDF 已从 arXiv 下载至本目录（`papers/`），点击"本地 PDF"即可直接阅读。每篇论文的详细解读见"解读"列。

| # | 论文（年份） | 一句话 | 作者单位 | 主要关键词 | 核心创新点 | 本地 PDF | 解读 | arXiv |
|---|---|---|---|---|---|---|---|---|
| 1 | SWE-agent: Agent-Computer Interfaces Enable Automated SWE（2024, NeurIPS） | 提出 ACI 概念：为 agent 重设计计算机接口 | Princeton Language and Intelligence, Princeton University | Agent-Computer Interface (ACI)、LM agent、software engineering、SWE-bench、interface design、ReAct | ① 首次系统论证"接口设计"对 LM agent 的决定性作用，提出 ACI 范式；② 为 agent 定制文件查看/编辑/搜索命令与 linting 护栏；③ SWE-bench pass@1 12.5%、HumanEvalFix 87.7%，显著超越非交互式 LM | [PDF](./2405.15793_SWE-agent.pdf) | [解读](./2405.15793_SWE-agent_解读.md) | [arXiv](https://arxiv.org/abs/2405.15793) |
| 2 | SWE-bench: Can Language Models Resolve Real-World GitHub Issues?（2023, ICLR 2024） | agentic 评测集与"评测驱动 harness 迭代"方法学 | Princeton University、Princeton Language and Intelligence、University of Chicago | benchmark、GitHub issues、software engineering、code generation、long context、execution-based evaluation | ① 首个从真实 GitHub issue + PR 构造的软件工程评测集（2,294 题 / 12 个 Python 仓库）；② 提出基于执行测试的自动评测（FAIL_TO_PASS / PASS_TO_PASS）；③ 揭示当时最强模型 Claude 2 仅解 1.96%，定义能力前沿 | [PDF](./2310.06770_SWE-bench.pdf) | [解读](./2310.06770_SWE-bench_解读.md) | [arXiv](https://arxiv.org/abs/2310.06770) |
| 3 | OpenHands: An Open Platform for AI Software Developers as Generalist Agents（2024, ICLR 2025） | 开源 harness 全栈标本：runtime + agent + 评测 | UIUC、CMU、Yale、UC Berkeley、Contextual AI、KAUST、ANU、HCMUT、Alibaba、All Hands AI | agent platform、sandbox runtime、CodeAct、event stream、multi-agent、benchmark integration、open-source | ① 开源通用 agent 平台，Runtime（Docker 沙箱）+ Agent（可插拔）+ Evaluation 三位一体；② 用事件流解耦 agent 循环与沙箱执行，支持轨迹回放与多 agent 协作；③ MIT 许可，15+ 任务/基准上的公开评测标本 | [PDF](./2407.16741_OpenHands.pdf) | [解读](./2407.16741_OpenHands_解读.md) | [arXiv](https://arxiv.org/abs/2407.16741) |
| 4 | The BrowserGym Ecosystem for Web Agent Research（2024/2025） | 把"环境"标准化为 gym 接口 | ServiceNow Research、Mila、Polytechnique Montréal、CMU、McGill、Tel Aviv University、Université de Montréal、iMean AI | web agent、benchmark、gym environment、AgentLab、observation/action space、unified LLM API | ① 提出统一 web agent 生态：BrowserGym（gym 式环境 + 统一观测/动作空间）+ AgentLab（实验管理与分析）；② 整合 MiniWoB/WebArena/WorkArena/WebLINX 等分散基准；③ 首次 6 个 SOTA LLM × 6 个基准的大规模横向对比 | [PDF](./2412.05467_BrowserGym.pdf) | [解读](./2412.05467_BrowserGym_解读.md) | [arXiv](https://arxiv.org/abs/2412.05467) |
| 5 | OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks（2024, NeurIPS） | OS 级 harness（截图+动作空间）研究平台 | The University of Hong Kong、CMU、Salesforce Research、University of Waterloo | benchmark、multimodal agent、real computer environment、GUI、execution-based evaluation、accessibility tree | ① 首个可在真实操作系统（Ubuntu/Windows/macOS）上运行的可扩展环境；② 369 个跨应用开放任务，每个含初始状态配置 + 执行式评测脚本；③ 揭示人类 72.36% vs 最佳模型 12.24% 的巨大差距，定位 GUI grounding 瓶颈 | [PDF](./2404.07972_OSWorld.pdf) | [解读](./2404.07972_OSWorld_解读.md) | [arXiv](https://arxiv.org/abs/2404.07972) |

**相关在线资源（无对应单一 arXiv 论文）**：

- Aider 官方文档与工程博客（edit format / repo map）：[aider.chat](https://aider.chat/)
- Anthropic — Building Effective Agents（workflow vs agent 模式目录）：[anthropic.com/engineering](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI — Practices for Governing Agentic AI Systems：[openai.com](https://openai.com/index/practices-for-governing-agentic-ai-systems/)
- Model Context Protocol 规范：[modelcontextprotocol.io](https://modelcontextprotocol.io/)
- Claude Code 官方 CLI：[github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)
- Terminal-Bench（终端任务评测基准）：[github.com/laude-institute/terminal-bench](https://github.com/laude-institute/terminal-bench)
