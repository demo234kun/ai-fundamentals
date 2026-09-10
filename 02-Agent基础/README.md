# 02 Agent 基础

## 第一部分：基本原理

### 0. 什么是 Agent

**定义（Lilian Weng 经典博客 "LLM Powered Autonomous Agents"）**：

```
Agent = LLM（大脑：推理与决策）
      + Planning（规划：任务分解、反思）
      + Memory（记忆：短期上下文 / 长期知识）
      + Tool Use（工具：调用外部世界）
```

与普通 chatbot 的本质区别：**模型不在单次调用中给出最终答案，而是在一个"感知 → 思考 → 行动 → 观察"的循环（loop）中迭代，直到任务完成**。模型的输出不是给人看的文本，而是给运行时（harness）解析的**结构化指令**（如 JSON tool call）。

### 1. ReAct：推理与行动交织的范式

**问题**：单纯 CoT（Chain-of-Thought）只会"想"不会"查"，事实性错误无法自我纠正；单纯 Action（传统强化学习代理）不会解释、难以调试。

**ReAct 方案**：交替生成 **Thought（推理轨迹）** 与 **Action（工具调用）**，把推理轨迹显式写出来：

```
Question: 雅加达现在气温比平均温度高多少？
Thought1: 我需要搜索雅加达现在的气温。
Action1: Search[雅加达 现在 气温]
Observation1: 30°C
Thought2: 现在我需要查雅加达的历史平均气温。
Action2: Search[雅加达 平均 气温]
Observation2: 27°C
Thought3: 30 - 27 = 3，答案高出 3 度。
Action3: Finish[3°C]
```

**工程要点**：
- 每次 Action 后控制权交还**运行时**，由运行时真正执行工具并把 Observation 塞回上下文——模型只负责"决定调用什么"。
- ReAct 是后续几乎所有 Agent 框架的底层循环：LangChain Agent、AutoGPT、OpenAI function calling 本质都是"模型声明动作 → 系统执行 → 回灌观察"。
- 局限：Thought 泄露用户隐私、长任务上下文爆炸、单步错误累积。

### 2. Planning：任务分解的艺术

**Plan-and-Execute（先规划后执行）**：
- **Planner**：一次性把任务拆成步骤列表 `[步骤1, 步骤2, ...]`，可以自身也是 LLM。
- **Executor**：逐条执行，每步结果回填给 Planner 决定是否重规划。
- 代表：BabyAGI、Plan-and-Solve、LangChain 的 Plan-and-Execute agent。
- 优点：长任务更稳（规划错误在开跑前就能暴露一部分）；缺点：环境变化后计划容易过期 → 引出"重规划（replanning）"。

**Tree of Thoughts（ToT）**：把"一条思维链"扩展成"思维树"，对每个中间状态采样多个分支，用 LM 自评 + 搜索算法（BFS/DFS）选择扩展路径。让模型能对弈、解谜题这类需要回溯的任务。

**Reflexion（反思）**：失败后不直接重试，而是让模型**把失败经验写成一段文字反馈**，存入 episodic memory，下一轮 prompt 带上这段反思再尝试。相当于给 Agent 加了一个"错题本"。这是"语言化自我改进"的关键思想。

### 3. Tool Use / Function Calling

**Toolformer**（Meta, 2023）：证明 LLM 可以**自监督地学会何时调 API**——用少样本提示让模型给语料插入 API 调用占位符，真正执行后用"插入后 loss 是否下降"做过滤，再拿过滤后的数据微调。无需人工标注的工具学习。

**现代 function calling 的工程标准**（OpenAI 2023.6 确立）：
```json
{
  "name": "get_weather",
  "description": "查询指定城市的天气",
  "parameters": {
    "type": "object",
    "properties": {
      "city": {"type": "string", "description": "城市名"},
      "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
    },
    "required": ["city"]
  }
}
```
- 模型输出结构化 JSON → **运行时（harness）负责 JSON Schema 校验、执行、异常回传**。
- 争议：JSON 是"最不 LLM-friendly"的格式，新一代协议转向更自然的语法（如 Anthropic 的工具块、MCP、或直接代码解释器 `exec`）。

**Code as Action（代码即动作）**：以代码为动作空间（CodeAct）——模型直接写一段 Python，由解释器执行。一次调用可组合多个工具（循环、分支、数据处理），信息密度远高于 JSON，代表：OpenAI Code Interpreter、smolagents 的 CodeAgent 默认模式。

**MCP（Model Context Protocol, Anthropic 2024.11）**：为工具/数据源定义**统一的服务端协议**（JSON-RPC over stdio/SSE）。Host（Claude Desktop / IDE）连接多个 MCP Server（文件系统、GitHub、数据库……），Agent 自动发现 server 提供的 tools/resources/prompts。价值：**工具生态标准化**，一次实现处处可用——本环境就接入了 github MCP。

### 4. Memory：短期与长期

| 记忆类型 | 载体 | 机制 |
|---|---|---|
| **短期（工作记忆）** | 当前上下文窗口 | 对话历史、工具返回、中间结果；受 token 上限约束 |
| **情景记忆（episodic）** | 向量库 + 文本日志 | 按相关性检索历史经历（Reflexion 的错题本、 Voyager 的技能档案） |
| **语义记忆（semantic）** | 外部知识库 | RAG / 知识图谱 |
| **程序记忆（procedural）** | SKILL.md / 代码 | 学会的技能本身，见模块 03 |

**MemGPT（后更名 Letta）**：把 OS 的虚拟内存分页思想搬到 LLM——当上下文装不下时，把"分页"到外部存储（文件/数据库），需要时取回，实现"拥有无限上下文"的操作系统隐喻。

### 5. Multi-Agent：多智能体协作

**设计模式**：
- **分工型**：Planner / Coder / Reviewer / Executor 各司其职（MetaGPT 的 SOP 思想：把软件公司角色映射为 agent）。
- **辩论型**：多个 agent 扮演正方/反方辩论，取共识输出（Du et al., "Improving Factuality via Multiagent Debate"）。
- **层级型**：Orchestrator-worker，主管拆任务派发给子代理，汇总结果（OpenAI Swarm / Claude Code subagent）。
- **群体型**：CAMEL 的角色扮演式协作、AutoGen 的可编程对话拓扑（群聊/顺序/嵌套）。

**何时值得多 Agent**：任务可自然分解、需要交叉验证（降低单 agent 幻觉）、不同子任务需要不同上下文/工具/权限隔离。**代价**：token 消耗成倍增长、通信协议设计复杂、错误会在 agent 间级联放大。

### 6. Agent 评估与可靠性

- **轨迹评估（Trajectory evaluation）**：不看最终答案，看"每一步动作是否合理"（AgentBench、WebArena 的做法）。
- **LLM-as-Judge**：用强模型给 agent 输出打分（MT-Bench、AgentBench）。
- **环境基准**：SWE-bench（真实 GitHub issue 修 bug）、WebArena/WorkArena（网页操作）、OSWorld（操作整个操作系统）、GAIA（通用助理）。

## 第二部分：主要论文

### 【核心论文】

1. **ReAct: Synergizing Reasoning and Acting in Language Models**（Yao et al., Google Brain + Princeton, 2022）——推理+行动交织范式，Agent 领域引用率最高之一。
2. **Toolformer: Language Models Can Teach Themselves to Use Tools**（Schick et al., Meta, 2023）——自监督工具学习。
3. **Chain-of-Thought Prompting Elicits Reasoning in LLMs**（Wei et al., Google, 2022）——CoT，一切推理型 Agent 的前置技术。
4. **Tree of Thoughts: Deliberate Problem Solving with Large Language Models**（Yao et al., 2023）——思维树 + 搜索。
5. **Reflexion: Language Agents with Verbal Reinforcement Learning**（Shinn et al., 2023）——语言化反思记忆。
6. **Plan-and-Solve Prompting / Plan-and-Execute Agents**（Wang et al., 2023 / LangChain, 2023）——先规划后执行范式。
7. **Generating Agents: Interactive Simulacra of Human Behavior**（Park et al., Stanford, 2023）——25 个 Agent 的"西部世界"小镇，记忆-反思-规划架构的经典实验。
8. **Voyager: An Open-Ended Embodied Agent with LLMs**（Wang et al., NVIDIA, 2023）——Minecraft 无限探索者，**自动沉淀可复用技能库**（见模块 03）。
9. **MemGPT: Towards LLMs as Operating Systems**（Packer et al., UC Berkeley, 2023）——虚拟内存式上下文管理。
10. **A Survey on LLM-based Autonomous Agents**（Wang et al., 2023）/ **The Rise and Potential of Large Language Model Based Agents**（Xi et al., 2023）——两篇体系化综述，建立领域知识框架。
11. **Model Context Protocol**（Anthropic, 2024.11，技术报告/规范文档）——工具协议标准化。
12. **The Landscape of Agent Evals**（OpenAI / METR 等, 2024-2025）——Agent 评测方法学综述（METR 的 task-completion time-horizon 度量）。

### 【应用领域：领域 + 主要创新】

1. **软件工程 + 从 issue 到 PR 全自动**：SWE-agent（ACI 概念）、OpenHands、AutoCodeRover、SWE-bench——Agent 在真实仓库修 bug 的最强竞技场。
2. **网页浏览 + 视觉-文本双模操作**：WebGPT / WebArena / Browser Use / Operator（OpenAI, 2025）——从文本接管网页到 GUI 截图操作的进化。
3. **计算机操作 + 统一 GUI 动作空间**：OSWorld / Claude Computer Use / UI-TARS——把"点哪、输什么"抽象成统一动作，跨应用完成办公任务。
4. **科学发现 + 闭环实验 Agent**：ChemCrow（化学合成规划+调真实 API）、CRISPR-GPT、AI Scientist（Sakana, 全自动写论文）——Agent 操作真实仪器/自动化脚本。
5. **个人助理 + 长任务持久化**：Devin（Cognition）、Manus（Monica, 2025）、OpenAI Deep Research——数小时级任务、异步执行、交付物导向。
6. **多智能体软件公司 + SOP 角色扮演**：MetaGPT / ChatDev——把"需求→设计→编码→测试"的公司流程编码为 agent 协作协议。
7. **数据库交互 + Text-to-SQL 执行闭环**：DIN-SQL / BIRD 榜单方案——schema 检索 + SQL 生成 + 执行反馈自纠错。
8. **机器人 + 语言规划控制**：SayCan（Google, 语言模型给技能打分 +  affordance 过滤）、Code as Policies、RT-2——LLM 作为机器人高层规划器。
9. **深度研究 + 检索循环**：Deep Research（OpenAI）/ GPT Researcher / Stanford STORM——多轮搜索-阅读-引用的 report-writing agent。
10. **安全与对齐 + Agent 红队**：AgentHarm / InjecAgent——间接提示注入（ poisoned web content 劫持 agent 行动）成为新攻击面。

## 第三部分：项目学习库（Coding 链接）

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| langchain-ai/langchain | 工具抽象、Agent 抽象层的历史与现状（建议读旧版 agent 代码理解概念演进） | https://github.com/langchain-ai/langchain |
| langchain-ai/langgraph | 生产级 Agent 编排：图状态机、人在环（HITL）、持久化 checkpoint | https://github.com/langchain-ai/langgraph |
| microsoft/autogen（现 AG2） | 多智能体可编程对话拓扑， conversational agents 奠基框架 | https://github.com/microsoft/autogen |
| microsoft/semantic-kernel | 微软的轻量 Agent SDK， planner + plugin 设计 | https://github.com/microsoft/semantic-kernel |
| crewAIInc/crewAI | 角色分工型多 agent 框架，crew/role/task 抽象最易上手 | https://github.com/crewAIInc/crewAI |
| huggingface/smolagents | 极简主义 Agent 库：代码即动作（CodeAct），代码量极小，**读源码首选** | https://github.com/huggingface/smolagents |
| OpenBMB/ChatDev | 虚拟软件公司多 agent 协作 | https://github.com/OpenBMB/ChatDev |
| geekan/MetaGPT | SOP 驱动的多 agent 框架 | https://github.com/geekan/MetaGPT |
| OpenAutoCoder/Agentless | 反直觉对照组：不用 agent、用检索+定位+修复流水线也能打 SWE-bench——理解"什么时候不需要 agent" | https://github.com/OpenAutoCoder/Agentless |
| All-Hands-AI/OpenHands | 开源版 Devin，完整的 SWE Agent 平台（代码+运行时+评测） | https://github.com/All-Hands-AI/OpenHands |
| modelcontextprotocol/servers | MCP 官方 server 集合，学协议实现的最佳样例库 | https://github.com/modelcontextprotocol/servers |
| anthropics/courses | Anthropic 官方课程：Agent 构建全教程（workshop 形式） | https://github.com/anthropics/courses |
| browser-use/browser-use | 用 LLM 操作真实浏览器 | https://github.com/browser-use/browser-use |
| stanford-oval/storm | 深度研究 agent：多视角检索写百科式报告 | https://github.com/stanford-oval/storm |

**动手路线建议**：
1. 用 `smolagents` 跑通 CodeAct agent（50 行内），打印完整 trajectory 理解 ReAct 循环。
2. 用 `langgraph` 搭一个 planner-executor-critic 三节点图，体验 checkpoint 与人审中断。
3. 用 `browser-use` 让 agent 自动完成一个真实网页任务（订机票信息采集），观察轨迹。
4. 读 `Agentless` 与 OpenHands 的方案对比，理解 agent 复杂度与收益的 trade-off。
