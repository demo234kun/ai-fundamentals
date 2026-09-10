# 03 Skill 基础

> Skill（技能）是 2024-2025 年 Agent 工程里最重要的概念之一：**把"做事的方法"从模型权重里、从 prompt 里，抽出来变成可沉淀、可复用、可分享的程序性记忆文件**。

## 第一部分：基本原理

### 0. Skill 是什么：程序性记忆的外置化

人类能力分为三类，Agent 同理：

| 能力 | 人类对应 | Agent 对应 |
|---|---|---|
| 陈述性知识（知道什么） | 事实记忆 | 模型权重 + RAG 检索 |
| 程序性知识（知道怎么做） | 骑车、做饭 | **Skill 文件** |
| 工具性能力 | 用手用工具 | function calling / MCP |

**定义**：Skill 是一个（通常以 `SKILL.md` 为核心的）**自包含指令包**，描述"在什么情况下，按什么步骤，用什么工具，完成某类任务"。模型在需要时加载它，按步骤执行，结果沉淀回 Skill 本身（改进步骤、修复错误）。

经典类比（Anthropic《Agent Skills》白皮书）：
> 工具教 Agent **"能做什么"**（capabilities），技能教 Agent **"该怎么做"**（procedures）。技能是"教Agent使用它的手，而不仅仅是给它手"。

### 1. Skill 与邻近概念的边界

```
Prompt          → 一次性、写在代码里的指令         （静态、无法演化）
System Prompt   → 全局行为准则，常驻上下文          （太大则挤占工作记忆）
Tool / Function → 模型可调用的能力单元              （what）
RAG             → 检索外部知识文本                  （know-what）
MCP             → 工具/数据源的连接协议             （管道）
Skill           → 可加载/卸载的流程性知识包          （know-how）★
```

- **Skill vs Tool**：tool 是代码接口；skill 是调用 tool 的**方法论**（何时调、按什么顺序、出错怎么办）。一个 skill 通常组合多个 tools。
- **Skill vs RAG**：RAG 检索的是"事实文本"；Skill 检索的是"操作手册"。但**底层机制相同**——都是按需加载进上下文（progressive disclosure）。
- **Skill vs Subagent**：subagent 是"另一个人"（独立上下文执行任务）；skill 是"一本手册"（主 agent 自己照着做）。

### 2. 核心机制一：渐进式披露（Progressive Disclosure）

Skill 文件通常分层组织，**只有在需要时才占用上下文**——这是解决"context is the new currency"的关键：

```
L1 索引（~100 token）：名字 + 一句话描述 + 触发词      ← 常驻系统提示的"技能目录"
L2 主体（~5k token）：SKILL.md 正文（步骤、要点）        ← 命中时加载
L3 资源（按需）：模板、脚本、示例、参考文档               ← 执行中再读
```

工程含义：
- 一个 Agent 可以注册几百个 skill，但任意时刻上下文里只有当前 skill 的 L2+L3。
- 触发方式：模型看到任务描述自己决定加载（靠 L1 描述与任务匹配），或运行时做语义检索（embedding 匹配 skill 描述）。
- 本环境（Kimi Work）的技能系统就是该机制的实例：`SKILL.md` + `Skill` 工具按需调用。

### 3. 核心机制二：从经验中自动沉淀（Voyager 思想）

Skill 的革命性不在于"手写手册"，而在于**自动生成与进化**：

**Voyager（NVIDIA, 2023）三步循环**：
1. **自动课程（Automatic Curriculum）**：根据当前状态与探索进度，LLM 提出"下一步该学什么"（越来越难的 Minecraft 目标）。
2. **技能库（Skill Library）**：学到的每个行为写成**可执行代码函数**存入向量库（描述 → 代码）。遇到新任务时检索相关技能复用/组合。
3. **迭代式提示机制**：环境反馈 + 执行错误自动反馈给模型重写代码，直到程序运行成功才入库。

此后"技能库"思想遍地开花：Agent 做完任务 → 把成功轨迹蒸馏成 skill → 下次同类任务直接调用。这是**程序性记忆的复利效应**。

### 4. SKILL.md 的工程规范（从各开源实践中归纳）

```markdown
# Skill 名称
## 概述（什么时候用这个技能，1-2 句）
## 触发条件 / 适用场景
## 前置条件（需要哪些工具、权限、输入）
## 步骤（编号、可执行、含出错分支）
## 输出格式 / 验收标准
## 注意事项（常见坑）
## 资源（scripts/、templates/、examples/ 相对路径）
```

**好 Skill 的特征**：
- **触发描述写满同义词**（L1 检索/匹配依赖描述覆盖度："图表/画图/可视化/plot/chart"）。
- **步骤是"检查清单"而非"散文"**——模型照着逐项打勾。
- **包含失败分支**（"如果 X 失败，则做 Y"），减少模型在死路上反复重试。
- **资源与主体分离**：大段模板/示例放附属文件，L2 只放骨架。
- **可验证**：结尾有明确的"完成判据"，可机器检查。

### 5. Skill 的组合与冲突

- **组合**：skill 可以引用 skill（"执行 X 前先调用 Y 技能"），形成技能图；运行时需做环检测与深度限制。
- **冲突**：多个 skill 都声称能处理同一任务时，需要路由机制（registry + 描述打分，如 `ccf-common` 这类共享治理 skill 所示）。
- **版本化**：skill 是代码资产，应进 git；改进后的 skill 要回归测试（用历史任务回放）。

### 6. Skill 生态的三代形态

| 代际 | 形态 | 代表 |
|---|---|---|
| 一代 | prompt 里的少样本示例 | few-shot prompting |
| 二代 | 手写 SKILL.md 文件库（人沉淀） | Claude Skills、Kimi Work skills、Cursor rules |
| 三代 | Agent 自动创建/进化技能（机沉淀） | Voyager、Agent Skills 自动生成、Cognition 的"记忆即代码" |

## 第二部分：主要论文

### 【核心论文】

1. **Voyager: An Open-Ended Embodied Agent with Large Language Models**（Wang et al., NVIDIA + Caltech + Stanford, 2023）——技能自动沉淀的开山之作：自动课程 + 技能库 + 迭代提示。
2. **Agent Skills**（Anthropic, 2024.12 白皮书）——把 SKILL.md 格式、渐进式披露、与工具/MCP 的关系确立为工程标准。
3. **Generative Agents**（Park et al., Stanford, 2023）——反思与记忆沉淀机制（observe→reflect→plan），技能的前身。
4. **Reflexion**（Shinn et al., 2023）——语言化经验沉淀，skill 生成的技术基础。
5. **ExpeL: LLM Agents Are Experiential Learners**（Zhao et al., 2023）——从成功/失败轨迹中**自动提取洞见（insight）**注入后续提示，Voyager 之后另一条经验沉淀路线。
6. **Text2Reward: Automated Dense Reward Function Generation for RL**（Xie et al., 2024）——把"技能"形式化为奖励函数自动生成，技能与 RL 的交叉点。
7. **CaMeL / Agentic Memory**（2024-2025 系列）——动态记忆图：技能不再静态检索，而是在使用中动态关联演化。
8. **A Survey on LLM-based Autonomous Agents**（Wang et al., 2023）——技能获取（skill acquisition）作为 agent 核心组件的体系化定位。

### 【应用领域：领域 + 主要创新】

1. **软件开发 + 从轨迹到可复用修复流程**：Cognition（Devin）公开分享——成功任务自动转成"playbook"；以及 SWE-smith / SWE-bench 社区沉淀的"修 bug 技能模板"。
2. **科研协作 + 全家桶技能治理**：CCFA 技能家族——路由、触发词注册表、任务模式、交接模式、隐私策略的共享治理，是"多 skill 工业化治理"的范本。
3. **办公软件 + 企业流程技能化**：Microsoft Copilot Studio 的"topics"与 Salesforce Agentforce 的"topics/actions"——企业把 SOP 编码为 agent 技能的标准商业实践。
4. **机器人 + 技能库与 affordance 结合**：SayCan 后继工作——预置技能原语库，LLM 负责组合调度（技能 = 机器人动作原语 + 使用说明）。
5. **游戏 + 开放世界技能涌现**：Voyager 后续（Odyssey、Ghost in the Minecraft）——技能库跨会话持久化使 agent 持续变强。
6. **数据分析 + 图表/报表技能模板化**：`kimi-design`、`kimi-excel` 等——把"设计规范/校验清单"沉淀为技能，新人 agent 零成本继承最佳实践。
7. **合规与安全 + 技能审计**：Anthropic Agent Skills 白皮书中的权限边界（skill 声明所需工具，运行时白名单校验）——技能成为新的安全审计单元。

## 第三部分：项目学习库（Coding 链接）

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| anthropics/skills | Anthropic 官方 skill 库：pdf/docx/xlsx/slides 等技能的 SKILL.md 写法教科书 | https://github.com/anthropics/skills |
| anthropics/courses | Anthropic 官方课程仓库，含 tool use / agent 章节 | https://github.com/anthropics/courses |
| MineDojo/Voyager | Voyager 官方代码：自动课程 + 技能库检索完整实现 | https://github.com/MineDojo/Voyager |
| Significant-Gravitas/AutoGPT（初代） | 早期"自动沉淀"冲动的历史标本（对照今昔） | https://github.com/Significant-Gravitas/AutoGPT |
| dair-ai/Prompt-Engineering-Guide | 从 prompt 到 skill 的演进全景（few-shot → CoT → agents → skills 章节） | https://github.com/dair-ai/Prompt-Engineering-Guide |
| modelcontextprotocol/servers | 看 MCP server 如何附带"使用说明"——工具与技能的交界实践 | https://github.com/modelcontextprotocol/servers |
| e2b-dev/awesome-ai-agents（社区合集） | Agent/技能生态的社区精选合集，找现成样例与资料 | https://github.com/e2b-dev/awesome-ai-agents |
| crewAIInc/crewAI 的 tools 目录 | 工具 + 说明文档绑定方式的最小实例 | https://github.com/crewAIInc/crewAI |

**动手路线建议**：
1. 通读 `anthropics/skills` 里 3 个 skill 的 SKILL.md（推荐 pdf、xlsx），体会 L1/L2/L3 分层。
2. 在 Minecraft 仿真或简化环境里复现 Voyager 的技能入库循环（MineDojo 提供环境）。
3. 把自己反复做的某类任务（如"生成周报"）手写成一个 SKILL.md，用三次迭代改进它——体会"沉淀"的价值。
4. 用 `skill-creator` 技能让 Agent 帮你把第 3 步的成果规范化。
