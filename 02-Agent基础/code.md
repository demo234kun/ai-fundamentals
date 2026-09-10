# 02 代码学习库：Agent 基础

> 按"先跑通 → 再读源码"的顺序排列。

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| huggingface/smolagents | 极简主义 Agent 库：代码即动作（CodeAct），代码量极小，**读源码首选** | https://github.com/huggingface/smolagents |
| langchain-ai/langgraph | 生产级 Agent 编排：图状态机、人在环（HITL）、持久化 checkpoint | https://github.com/langchain-ai/langgraph |
| langchain-ai/langchain | 工具抽象、Agent 抽象层的历史与现状 | https://github.com/langchain-ai/langchain |
| microsoft/autogen（现 AG2） | 多智能体可编程对话拓扑，conversational agents 奠基框架 | https://github.com/microsoft/autogen |
| microsoft/semantic-kernel | 微软的轻量 Agent SDK，planner + plugin 设计 | https://github.com/microsoft/semantic-kernel |
| crewAIInc/crewAI | 角色分工型多 agent 框架，crew/role/task 抽象最易上手 | https://github.com/crewAIInc/crewAI |
| OpenBMB/ChatDev | 虚拟软件公司多 agent 协作 | https://github.com/OpenBMB/ChatDev |
| geekan/MetaGPT | SOP 驱动的多 agent 框架 | https://github.com/geekan/MetaGPT |
| OpenAutoCoder/Agentless | 反直觉对照组：不用 agent 也能打 SWE-bench——理解"什么时候不需要 agent" | https://github.com/OpenAutoCoder/Agentless |
| All-Hands-AI/OpenHands | 开源版 Devin，完整的 SWE Agent 平台（代码+运行时+评测） | https://github.com/All-Hands-AI/OpenHands |
| modelcontextprotocol/servers | MCP 官方 server 集合，学协议实现的最佳样例库 | https://github.com/modelcontextprotocol/servers |
| anthropics/courses | Anthropic 官方课程：Agent 构建全教程（workshop 形式） | https://github.com/anthropics/courses |
| browser-use/browser-use | 用 LLM 操作真实浏览器 | https://github.com/browser-use/browser-use |
| stanford-oval/storm | 深度研究 agent：多视角检索写百科式报告 | https://github.com/stanford-oval/storm |

**动手路线**：
1. 用 `smolagents` 跑通 CodeAct agent（50 行内），打印完整 trajectory 理解 ReAct 循环。
2. 用 `langgraph` 搭一个 planner-executor-critic 三节点图，体验 checkpoint 与人审中断。
3. 用 `browser-use` 让 agent 自动完成一个真实网页任务，观察轨迹。
4. 读 `Agentless` 与 OpenHands 的方案对比，理解 agent 复杂度与收益的 trade-off。
