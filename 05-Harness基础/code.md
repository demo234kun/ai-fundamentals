# 05 代码学习库：Harness 基础

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| All-Hands-AI/OpenHands | 开源 harness 全栈标本：runtime 沙箱、agent 循环、trajectory 回放 | https://github.com/All-Hands-AI/OpenHands |
| SWE-agent/SWE-agent | ACI 论文官方实现，读它理解"为 agent 设计接口" | https://github.com/SWE-agent/SWE-agent |
| Aider-AI/aider | 最精炼的生产级编码 harness：edit format、repo map、git 集成 | https://github.com/Aider-AI/aider |
| OpenInterpreter/open-interpreter | 代码解释器式 harness 的社区先驱 | https://github.com/OpenInterpreter/open-interpreter |
| modelcontextprotocol/python-sdk | MCP 协议 SDK——harness 的工具总线标准 | https://github.com/modelcontextprotocol/python-sdk |
| modelcontextprotocol/servers | MCP 官方 server 目录 | https://github.com/modelcontextprotocol/servers |
| langchain-ai/langgraph | 把 harness 组件（状态图、checkpoint、human-in-the-loop）做成 SDK | https://github.com/langchain-ai/langgraph |
| laude-institute/terminal-bench | 终端任务评测基准：比较不同 harness/模型在同一环境下的能力 | https://github.com/laude-institute/terminal-bench |
| browser-use/browser-use | 浏览器型 harness：DOM 提取+动作空间设计 | https://github.com/browser-use/browser-use |
| OpenPipe/ART（Agent Reinforcement Trainer） | 用可验证奖励在真实 harness 轨迹上训练 agent | https://github.com/OpenPipe/ART |
| anthropics/claude-code | 闭源但可观察：日志与公开 system prompt 逆向学习工业 harness | https://github.com/anthropics/claude-code |

**动手路线**：
1. 用 200 行 Python 手写一个最小 harness：system prompt + 3 个工具（读文件/跑命令/搜索）+ ReAct 循环 + 步数上限。
2. 对比用同一个模型驱动：裸 API vs 你的 mini-harness vs Claude Code/Aider，在同一批任务上比较成功率。
3. 给 mini-harness 加 compaction 与计划模式，体会状态管理的难度跃升。
4. 读 OpenHands 的 `agenthub` 与 runtime 代码，看"沙箱执行"如何与 agent 循环解耦。
