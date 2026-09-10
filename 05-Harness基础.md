# 05 Harness 基础（Agent 运行时 / 脚手架）

> Harness（挽具/挽具架）是当前 agentic AI 工程圈对**"包裹在模型外面的整个运行时系统"**的统称：模型是发动机，harness 是车架、传动、仪表盘和交通规则。Claude Code、Kimi Work、Codex、Aider 本质是**同一个大脑（LLM）配上不同的 harness**。

## 第一部分：基本原理

### 0. 为什么 harness 是"另一半系统"

同一个 GPT-x / Claude / Kimi 模型：
- 裸 API 调用：只会续写文本；
- 装进 Claude Code / Kimi Work：能读写文件、跑命令、浏览网页、管理长任务。

能力差异几乎全部来自 harness。研究结论（多个 benchmark 的经验规律）：**在 agentic 任务上，harness 的改进收益常常大于换更强的模型**。SWE-bench 早期榜首靠 harness 工程而非模型尺寸，是著名例证。

### 1. Harness 的五大核心组件

```
┌─────────────────────────────────────────────────────┐
│ ① Context Assembler（上下文组装器）                    │
│    系统提示 + 技能索引 + 工具 schema + 记忆 + 对话历史   │
├─────────────────────────────────────────────────────┤
│ ② Tool Runtime（工具运行时）                           │
│    工具注册表 / MCP / 权限与审批 / 错误回传格式          │
├─────────────────────────────────────────────────────┤
│ ③ Agent Loop（主循环）                                │
│    模型输出 → 解析 → 执行 → 观察回灌 → 循环直至终止条件   │
├─────────────────────────────────────────────────────┤
│ ④ State & Memory（状态与记忆）                         │
│    会话持久化 / 上下文压缩(compaction) / 跨会话记忆       │
├─────────────────────────────────────────────────────┤
│ ⑤ Delivery & Safety（交付与安全）                      │
│    产物落盘 / 计划模式 / 人工确认闸门 / 审计日志          │
└─────────────────────────────────────────────────────┘
```

### 2. ① 上下文组装：给模型一个"工作台"

- **系统提示（System Prompt）**：身份、行为准则、输出契约、环境描述（如 Kimi Work 的系统提示会注入当前时间、工作区路径、技能索引、Canvas 状态）。
- **工具 schema 注入**：每个工具的 JSON Schema 常驻上下文——工具一多就爆炸，所以有**工具检索/分页加载**与 **MCP 按需发现** 的设计。
- **运行时元数据注入**：当前时间戳、平台、权限模式（如 auto permission mode）——这些是"从环境中来的系统消息"，模型必须知道自己在哪、能干什么。
- **上下文预算管理**：上下文是稀缺货币。好的 harness 会精确计算 token，预留输出空间，避免"塞满导致截断"。

### 3. ② 工具运行时：能力=接口+权限+错误契约

- **三层工具来源**：内置工具（文件/终端/搜索）→ 技能（见模块 03）→ MCP 外部 server（github、tianyancha、yahoo_finance……）。
- **权限模型**：只读默认、写操作需确认、不可逆操作（删除/外发）必须人工闸门；**auto 模式**是"预授权清单"而非无限制。
- **错误回传契约**：工具失败返回结构化错误（exit code、stderr），模型据此自纠错——harness 决定给模型多少观察信息（太少=瞎修，太多=上下文爆炸）。

### 4. ③ Agent 循环：ReAct 的工程化

```
loop:
  1. 组装上下文 → 调用模型
  2. 解析输出：文本？工具调用？（function-call 协议或代码块协议）
  3. 校验：JSON Schema 校验、参数类型、权限检查
  4. 执行工具，捕获 stdout/结果/异常
  5. 以"工具结果消息"回灌上下文
  6. 检查终止条件：任务完成声明 / 步数上限 / token 预算 / 用户中断
```

**工程细节决定成败**：
- **并行工具调用**：无依赖的多个调用并行执行（本环境同响应内多调用即此设计）。
- **流式与中断**：长任务需可取消、可追问。
- **计划模式（Plan Mode）**：先收集信息、产出计划、人审后再执行——把高风险动作从循环里摘出来。
- **子代理（subagent）**：把大任务切块给独立上下文的 worker，只回传结论——控制主上下文污染。

### 5. ④ 状态与记忆：跨会话的连续性

- **Compaction（压缩）**：上下文将满时，把早期历史总结成摘要替换原文——所有长任务 harness 的标配（Claude Code、Codex 均有）。
- **会话持久化**：wire/session 文件落盘，重启续跑。
- **注入式记忆**：如本环境的 `<meta>` 时间戳、记忆文件、Canvas 状态——**系统注入的上下文必须被当作"输入"而非"用户指令"**（防提示注入的第一原则）。
- **交付物落盘纪律**：大产物（报告/表格/图表）边做边存盘，而不是攒到最后一次输出。

### 6. ⑤ 交付与安全

- **产物验证**：网页要真打开看一眼、表格要 reconcile 数字——"交付验证"是 harness 级职责而非模型职责。
- **审计**：每次工具调用可回放（OpenHands 的 trajectory、本环境的 run transcript）。
- **防注入**：网页/文档内容中的恶意指令不能劫持 agent（间接提示注入是 harness 必须设防的攻击面）。

### 7. 评价一个 harness 的清单

1. 上下文管理是否聪明（注入了什么、何时压缩）？
2. 工具协议是否统一（MCP？）、权限是否最小化？
3. 循环是否有终止保证（步数/预算/中断）？
4. 子代理与并行调度是否一等公民？
5. 状态是否持久可恢复？
6. 交付验证是否内建？
7. 人机协作档位是否清晰（auto / plan / 逐步确认）？

## 第二部分：主要论文

### 【核心论文】

1. **SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering**（Yang et al., Princeton, 2024）——明确提出 **ACI（Agent-Computer Interface）** 概念：为 agent 重新设计的计算机接口，比为人设计的 GUI 更重要；harness 工程的奠基论文。
2. **SWE-bench / SWE-bench Verified**（Jimenez et al., 2023/2024）——agentic 评测集与"评测驱动 harness 迭代"方法学。
3. **OpenHands: An Open Platform for AI Software Developers as Generalist Agents**（Wang et al., 2024）——开源 agent 平台：runtime（沙箱）+ agent + 评测三位一体，是研究 harness 架构的公开标本。
4. **Aider: AI Pair Programming in Your Terminal**（Paul Gauthier, 技术报告/blog 系列, 2024）——repo map、edit format 设计等纯 harness 技巧显著提升编码模型效果。
5. **Don't Build Multi-Agents? / Multi-Agent 争论**（Anthropic "Building Effective Agents"、OpenAI "Practices for Governing Agentic AI Systems", 2024-2025）——**编排模式选型**：workflow（预定义流水线） vs agents（自主循环），以及 router/orchestrator-worker/evaluator-optimizer 等模式目录。
6. **Chain-of-Thought Monitoring / agent safety evals**（Anthropic & Redwood, 2024-2025）——监控 agent 思维链进行安全审计，harness 层的对齐手段。
7. **Model Context Protocol**（Anthropic, 2024.11）——工具接入协议标准化，可视为"harness 插件总线"的工业标准。
8. **The BrowserGym Ecosystem for Web Agent Research**（2024）——把"环境"标准化为 gym 接口，harness 与环境的解耦范式。
9. **OSWorld / WorkArena / WindowsAgentArena**（2024）——OS 级 harness（截图+动作空间）的研究平台。
10. **Claude Code / Kimi Work / Codex 官方系统提示与工程博客**（2025，公开流出的 system prompt 与官方 writeup）——研究工业级 harness 提示词设计的稀缺一手材料。

### 【应用领域：领域 + 主要创新】

1. **软件工程 + ACI 重设计**：SWE-agent / OpenHands / Aider / Claude Code——文件编辑格式（diff/search-replace）、repo map、测试回环，编码 harness 的"四大发明"。
2. **通用助理 + 会话即产物**：Kimi Work / Claude Desktop——会话内直接生成可交付文件、定时任务（Automation）、看板（Canvas），把 harness 做成"操作系统"而非"插件"。
3. **浏览器自动化 + 真实登录态**：Kimi WebBridge / Browser Use / Skyvern——用用户真实浏览器会话，绕过验证码与登录态问题。
4. **数据分析 + 代码解释器内环**：OpenAI Code Interpreter / Jupyter 型 harness——生成代码→沙箱执行→把图表/表格回灌给模型，数据分析的标准 harness。
5. **终端 CLI + 最小 harness**：Aider / Codex CLI / Gemini CLI——证明"文件读写+shell+git"三件套足以构成强 harness，复杂性主要在上下文组织。
6. **科研 + 实验闭环 harness**：AutoResearcher / ChemCrow——harness 管理实验设备 API 与文献库，安全闸门更严格。
7. **移动端/桌面 GUI + 无障碍树接口**：AppAgent / Mobile-Agent——用 accessibility tree（而非截图）作为观察空间，token 效率与稳定性兼得。
8. **安全红队 + 间接注入防御**：AgentHarm / InjecAgent 及防御方案（内容隔离标签、指令层级、工具输出消毒）——harness 是最后防线。
9. **多模型路由 harness**：LiteLLM / OpenRouter 型网关——按任务自动路由到不同模型/价格档，harness 承担"模型调度器"角色。
10. **可观测性 + Agent  tracing**：LangSmith / Arize Phoenix / OpenTelemetry GenAI 语义约定——把每次模型调用与工具调用全链路记录，harness 的"黑匣子"。

## 第三部分：项目学习库（Coding 链接）

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| All-Hands-AI/OpenHands | 开源 harness 全栈标本：runtime 沙箱、agent 循环、trajectory 回放 | https://github.com/All-Hands-AI/OpenHands |
| SWE-agent/SWE-agent | ACI 论文官方实现，读它理解"为 agent 设计接口" | https://github.com/SWE-agent/SWE-agent |
| Aider-AI/aider | 最精炼的生产级编码 harness：edit format、repo map、git 集成 | https://github.com/Aider-AI/aider |
| OpenInterpreter/open-interpreter | 代码解释器式 harness 的社区先驱 | https://github.com/OpenInterpreter/open-interpreter |
| modelcontextprotocol/python-sdk / modelcontextprotocol/servers | MCP 协议 SDK 与 server 目录——harness 的工具总线标准 | https://github.com/modelcontextprotocol/python-sdk |
| langchain-ai/langgraph | 把 harness 组件（状态图、checkpoint、human-in-the-loop）做成 SDK | https://github.com/langchain-ai/langgraph |
| laude-institute/terminal-bench | 终端任务评测基准：比较不同 harness/模型在同一环境下的能力 | https://github.com/laude-institute/terminal-bench |
| browser-use/browser-use | 浏览器型 harness：DOM 提取+动作空间设计 | https://github.com/browser-use/browser-use |
| OpenPipe/ART（Agent Reinforcement Trainer） | 用可验证奖励在真实 harness 轨迹上训练 agent（模块 01 与本模块的交汇） | https://github.com/OpenPipe/ART |
| anthropics/claude-code（官方 CLI） | 闭源但可观察：用 `--verbose`、日志与公开 system prompt 逆向学习工业 harness | https://github.com/anthropics/claude-code |
| 本机：Kimi Work | 你正在使用的 harness 实例：技能系统、Automation、Canvas、权限模式——对照本文件逐条观察本环境的对应设计 | 本地应用 |

**动手路线建议**：
1. 用 200 行 Python 手写一个最小 harness：system prompt + 3 个工具（读文件/跑命令/搜索）+ ReAct 循环 + 步数上限——一周内你会理解 90% 的工业 harness 设计决策。
2. 对比用同一个模型驱动：裸 API vs 你的 mini-harness vs Claude Code/Aider，在同一批任务上比较成功率。
3. 给 mini-harness 加 compaction 与计划模式，体会状态管理的难度跃升。
4. 读 OpenHands 的 `agenthub` 与 runtime 代码，看"沙箱执行"如何与 agent 循环解耦。

---

## 五大模块的交汇图（收尾）

```
                    ┌─────────────────────────────┐
                    │      Harness（运行时）        │  ← 模块05：承载一切
                    │  上下文组装/工具运行时/循环/   │
                    │  状态记忆/交付安全             │
                    └──────┬───────────┬──────────┘
            调用模型时挂载  │           │ 执行任务时调用
                    ┌──────┴───┐   ┌───┴──────────┐
                    │ 模型能力  │   │ Agent（02）   │
                    │ 由模块 01 │   │ ReAct/规划/工具│
                    │ 炼出来   │   └──────┬───────┘
                    └──────────┘          │ 经验沉淀
                       ▲                  ▼
                 SFT/DPO/KTO/GRPO    Skill（03）
                 对齐与推理能力        程序性记忆
                       │
                    RAG（04）：给模型与 agent 外挂可溯源知识
```
