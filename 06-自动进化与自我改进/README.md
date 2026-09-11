# 06 自动进化与自我改进（Self-Evolving Agents / RSI）

> 本模块整理"自动进化"（Self-Evolving / Recursive Self-Improvement, RSI）方向。核心参照是 2026.9 发布的闭环 RSI 三件套：[Aspire](https://arxiv.org/abs/2608.31111) / [S³Gym](https://arxiv.org/abs/2608.31100) / [HarnessDev](https://arxiv.org/abs/2609.01437)（项目页：[self-developing-agents.github.io](https://self-developing-agents.github.io/)）。

## 第一部分：基本原理

### 0. 什么是"自动进化"

先区分三个常被混用的层级（Philschmid 的标尺比喻）：

| 层级 | 循环里什么在变 | 例子 | 验证者（Verifier） |
|---|---|---|---|
| **迭代 Iteration** | 只有输出变，系统不变 | agent 改代码、跑测试、完成当前任务 | 外部固定 |
| **自我改进 Self-Improvement** | **系统持久地变**（加工具、存技能、改 harness、改权重） | Voyager 技能库、Continual Harness、SIA | 外部固定 |
| **递归自我改进 RSI** | 系统变，**评判标准本身也提升** | 系统既会找改进、又会改进"怎么评判改进" | 随循环变硬（但不可被刷） |

当前绝大多数系统停在第二层，而且依赖一个 **Golden Verifier**（外部、诚实、不可被 agent 篡改的验证者）告诉它"改对了没有"。三件套论文的核心论点：**真正的闭环 RSI 有半圈是大家没测的**——在 verifier 之外，还有三个更根本的问题。

### 1. 闭环 RSI 的三问（本模块主线）

```
        ① TARGET 目标形成
        "宽泛模糊的目标，如何变成具体可训练的改进动作？"
        （Aspire：给"提升数学推理"这种模糊目标，不告诉下游任务）
                    │
        ② EXPERIENCE 经验整合
        "没有老师的情况下，能否判断自己的行为好坏并学到东西？"
        （S³Gym：自测、自评、自改进能否闭环？）
                    │
        ③ SYSTEM 系统集成
        "改进能否持久存在——写进代码的机制，运行时真的在起作用吗？"
        （HarnessDev：自建 harness 并用执行反馈进化它）
                    │
        ═══ 半圈到此为止；闭环还差"评判者本身进化" ═══
```

### 2. 三件套论文详解

#### ① Aspire —— 目标形成（Target Operationalization）

- **设定**：给 agent 一个宽泛能力目标（如"改进数学推理"），不透露下游任务与最终评测；agent 自己要决定学什么、怎么学（选数据、选更新方法、训 checkpoint）。
- **关键机制**：密封评测（sealed/hidden evaluation）+ 最小交互环境（只暴露语义动作：注册数据、训练、验证、分叉、终止；隐藏后端）；每条候选记录 lineage（父 checkpoint、数据、provenance），**晋级是显式的轮间操作，绝不在轮内自动替换**。
- **核心发现**：
  - 30 个"配置×目标"格子中 28 个产出了 checkpoint，只有两个超过基座，**只有一个通过了留存阈值**——"跑完更新循环 ≠ 改进了目标能力"。
  - 模糊目标使 agent 把大量时间花在"解读目标、选代理指标（proxy）"上（决策思考时间 +35 分钟/对，GPU 训练评估时间 −76 分钟），LoRA 使用率从 24.1% 飙到 89.8%——**搜索分布变了，目标还没被验证**。
- **启示**：目标操作化（把模糊意图翻译成可训练目标 + 验证标准）是自主后训练缺失的一轴；**必须用 agent 看不见的评测来检验它选的靶子**。

#### ② S³Gym —— 经验整合（Self-Test → Self-Judge → Self-Improve）

- **设定**：7 个可验证文本游戏（Chess、Minesweeper、Nullify、Tetris、Snake、PvZ、Trust Evolution），探索期 verifier 奖励对 agent 隐藏；循环分五阶段：Explore → Judge → Consolidate → Update → Evaluate。
- **三种经验通路对比**（对应"经验以什么形式留下来"）：
  1. **History ICL**：原始轨迹直接进 prompt——细节全但吃上下文；
  2. **Summary Memory**：压缩成"要保留的策略 / 要避免的错误"——省 token 但抽象可能错；
  3. **Parameter Training**：高分动作做成 SFT 数据更新权重——最持久。
- **核心发现**：
  - **没有一种通路在所有环境赢**：Summary Memory 在 Nullify/Tetris/Trust 领先，History ICL 在 Minesweeper/Snake/PvZ 领先，Chess 打平——换归一化指标排行榜还会重排。
  - **自评几乎预测不了下一步收益**（ρ = −0.010 和 −0.018）——自评好 ≠ 真的改进了。
  - 参数更新不稳定：Trust 上 20 个 checkpoint 中 18 个高于基线；PvZ 却从 23 跌到 6 后卡住不动。
- **启示**：经验通路要**按环境选、在 held-out 决策上确认**；"自评可信度"本身就是需要被验证器度量的对象。

#### ③ HarnessDev —— 系统集成（Build → Evolve → Persist）

- **设定**：让 LLM **从零创建**可运行的 agent harness（Creation 阶段），再用下游执行反馈**进化**它（Evolution 阶段）；被评测的是 harness 本身（冻结的版本在 held-out 任务上跑），不只是任务输出。
- **评测设计**：可见分数集（反馈集）vs 从未回流的 held-out 集（SWE-Pro 630 任务 + Terminal-Bench 89 任务）；还用固定 Gemini 执行器跑对照（检验"换执行模型改进还在吗"）。
- **核心发现**：
  - 18 个创建的 Code harness 全部能跑，但 **11/18 声明了 State 类、1/18 有状态保存接口、1/18 有周期 checkpoint——而 26,679 条任务轨迹中 checkpoint 事件为 0**：机制存在于代码，却没有活过运行时。
  - 64 次版本切换中，编辑集中在执行流/工具/生命周期恢复（58/37/17），很少动状态管理（4）和独立验证器（16 中含部分）——**agent 爱改"流"，不爱改"记忆与验证"**。
  - held-out 泛化弱：可见分与 held-out 分在 64 次切换中只有 34 次一致；9 个自报"最终版"里只有 2 个是 held-out 最优；固定执行器下 4 条 lineage 只有 1 条（Opus）不退化。
- **启示**：**给每次变更做版本化 + 回滚，只在 held-out 上验证过的改动才保留**——"知道哪些该留"比"持续变"重要。

### 3. 三件套共同揭示的三大失败模式

| 断裂点 | 表现 | 对策（论文给出） |
|---|---|---|
| 目标错了（Target） | checkpoint 在变好，但都低于基座；改进没被留存 | 用 agent 看不见的隐藏评测检验目标 |
| 经验不迁移（Experience） | 自评与真实收益无关（ρ≈0）；没有通吃的方法 | 按环境选通路，在 held-out 决策上确认 |
| 机制不存活（System） | 代码里有状态机制，运行时从不触发；held-out 上退化 | 版本化 + 回滚 + 固定执行器复测 |

### 4. FDE 类比：什么才值得留下

论文用"前置部署工程师（Forward-Deployed Engineer）"作类比，五步：

1. **Platform**：从通用平台出发；2. **Discover**：找到真实客户问题；3. **Deliver**：交付可用的解；4. **Learn**：提炼可复用的模式；5. **Compound**：把验证过的能力整合回去。
- 最终检验是**复用**：下一个项目能不能直接用。对应到 agent：一次"改进"只有当它成为**后续任务可用的持久能力**（权重/记忆/harness 里的活机制）才算数。

### 5. 与本知识库其他模块的关系

- **模块 03（Skill）**：Voyager 式"执行成功才入库"就是"留存验证"的最简形式；S³Gym 证明经验通路选择本身需要基准。
- **模块 05（Harness）**：HarnessDev 是"评测驱动 harness 迭代"（SWE-agent 路线）的严格化——把反馈集/held-out 集分离，把"运行时使用率"纳入评估。
- **模块 01（RL/GRPO）**：RLVR 的"可验证奖励"在这里被拆穿一半——奖励可见时刷分容易，难的是目标选择与改进留存。

## 第二部分：主要论文

### 【核心论文】（三件套，PDF 已下载至 papers/）

1. **Aspire: Can Models Self-Evolve from Vague Goals?**（2026.9）——目标形成基准；30 格仅 1 格留存增益。
2. **S³Gym: Can LLMs Turn Self-Testing and Self-Judging into Self-Improvement?**（2026.9）——经验整合基准；三通路无通吃、自评 ρ≈0。
3. **HarnessDev: Can LLMs Create and Evolve Their Own Agent Harness?**（2026.9）——系统集成基准；机制存活率与 held-out 泛化。

### 【应用领域：领域 + 主要创新】（2025-2026 代表工作，按 lobehub/awesome-rsi 谱系整理）

1. **模型权重级进化 + 自动化后训练**：PostTrainBench（一块 H100 十小时自主后训练）、RSIBench-Data（固定栈下迭代数据策略）、AI4AI-Bench（重写训练算法）——Aspire 的直接前身。
2. **Harness 级进化 + 观测驱动**：Agentic Harness Engineering、AutoHarness（145 个 TextArena 游戏合成 harness）、Continual Harness（单轨迹在线改 prompt/子 agent/技能）、EvoHarness-RL（Belief/Progress/Experience 状态演化）。
3. **技能/记忆级进化**：Evo-Memory、EvoFSM（有限状态工作流编辑）、SkillRise（RL 交替策展技能文档）、SkillLearnBench（外部反馈支持持续增益、纯自反馈导致漂移）。
4. **评测基准类**：HarnessOpt-Bench（种子 harness + 梯度反馈优化）、Evo-Bench（固定执行器改 CodeAct 种子）、PAST-Bench（存的经验是否真的帮到后续 episode）、RSI-Bench（六轴自改深度框架）。
5. **具身进化**：ASPIRE（机器人技能库，注意与本文 Aspire 同名不同团队）、ENPIRE（真机 autoresearch 闭环）、MineEvolve（Minecraft 经验转技能+护栏）。
6. **安全与对齐**：递归自我改进的 verifier 安全性（"评判者本身能否被刷"）是 RSI 从半圈到闭环的关键风险面。

## 第三部分：项目学习库（Coding 链接）

见 [code.md](./code.md)。

---

> 更新时间：2026-09-11。三件套为 9 月 1 日新发布论文，解读基于项目页与 arXiv 全文整理，个别数字以论文最终版为准。
