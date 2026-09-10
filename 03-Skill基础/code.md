# 03 代码学习库：Skill 基础

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| anthropics/skills | Anthropic 官方 skill 库：pdf/docx/xlsx/slides 等技能的 SKILL.md 写法教科书 | https://github.com/anthropics/skills |
| MineDojo/Voyager | Voyager 官方代码：自动课程 + 技能库检索完整实现 | https://github.com/MineDojo/Voyager |
| anthropics/courses | Anthropic 官方课程仓库，含 tool use / agent 章节 | https://github.com/anthropics/courses |
| dair-ai/Prompt-Engineering-Guide | 从 prompt 到 skill 的演进全景 | https://github.com/dair-ai/Prompt-Engineering-Guide |
| modelcontextprotocol/servers | 看 MCP server 如何附带"使用说明"——工具与技能的交界实践 | https://github.com/modelcontextprotocol/servers |
| e2b-dev/awesome-ai-agents | Agent/技能生态的社区精选合集 | https://github.com/e2b-dev/awesome-ai-agents |
| crewAIInc/crewAI 的 tools 目录 | 工具 + 说明文档绑定方式的最小实例 | https://github.com/crewAIInc/crewAI |

**动手路线**：
1. 通读 `anthropics/skills` 里 3 个 skill 的 SKILL.md（推荐 pdf、xlsx），体会 L1/L2/L3 分层。
2. 在 Minecraft 仿真或简化环境里复现 Voyager 的技能入库循环（MineDojo 提供环境）。
3. 把自己反复做的某类任务（如"生成周报"）手写成一个 SKILL.md，用三次迭代改进它——体会"沉淀"的价值。
