# 01 微调与强化学习（LoRA / SFT / RLHF / DPO / KTO / GRPO）

## 第一部分：基本原理

### 0. 总览：从基座模型到可用模型

```
预训练基座模型 (Base Model)
      │  ① SFT 监督微调：学会"听指令、按格式回答"
      ▼
指令模型 (Instruct Model) ──────② LoRA/QLoRA：低成本微调的技术手段（可作用于 ①③④ 任意环节）
      │
      │  ③ 偏好对齐：让回答"合人意"而非"像人话"
      │     路线A：RLHF（奖励模型 + PPO）
      │     路线B：DPO / KTO / IPO / ORPO（免 RL 的直接偏好优化）
      ▼
对齐模型 (Aligned Model)
      │
      │  ④ 推理期强化学习 RLVR：用可验证奖励练"推理能力"
      │     GRPO / RLOO / REINFORCE++（DeepSeek-R1 路线）
      ▼
推理模型 (Reasoning Model, e.g. R1 / o1)
```

### 1. SFT（Supervised Fine-Tuning，监督微调）

**原理**：用"指令 → 理想回答"的成对样本 `(x, y*)`，以标准自回归交叉熵损失继续训练基座模型：

```
L_SFT = - Σ_t log P_θ(y*_t | x, y*_<t)
```

**数据格式**：通常是 chat template 包装的多轮对话 JSON：

```json
{"messages": [
  {"role": "system", "content": "你是一个代码助手"},
  {"role": "user", "content": "用 Python 写快排"},
  {"role": "assistant", "content": "def quicksort(...)"}
]}
```

**工程要点**：
- **只算 assistant 部分的 loss**（completion-only loss），对 system/user token 做 mask，避免模型学"怎么提问"。
- **灾难性遗忘**：全参微调会损伤基座通用能力 → 引出 LoRA / 小学习率 / 早停等方案。
- **数据质量 >> 数据数量**：LIMA 论文提出 "Less Is More for Alignment"，约 1000 条高质量样本即可激发出指令能力。
- **超参经验**：lr 1e-5 ~ 2e-5（全参）/ 1e-4 ~ 2e-4（LoRA），epoch 2~3，packing 提升吞吐，NEFTune 噪声嵌入可提升泛化。

### 2. LoRA（Low-Rank Adaptation，低秩适配）

**原理**：冻结原权重 `W₀`，把权重增量约束为低秩分解 `ΔW = BA`，其中 `B ∈ R^(d×r)`，`A ∈ R^(r×k)`，秩 `r << min(d,k)`（常用 8/16/64）。

```
h = W₀x + ΔWx = W₀x + BAx        （前向时 BA 乘出来合并，推理零额外延迟）
```

- **初始化**：`A` 用高斯随机初始化，`B` 初始化为 0 → 训练起点 `ΔW = 0`，等价于原模型。
- **缩放**：`ΔW` 实际生效为 `(α/r)·BA`，`α` 常用 16/32/64。`r` 调大时通过 `α/r` 保持更新量级稳定。
- **为什么有效**：预训练权重矩阵的"任务适配"天然是低秩的——改变模型行为不需要动所有自由度，只需要在关键的低维子空间里调整。
- **落在哪里**：只挂在注意力矩阵（q/k/v/o）叫 LoRA；连 MLP 层一起挂叫 "full LoRA"；几乎所有实践都同时挂 qkv+o。

**QLoRA**（LoRA 的量化版）：
- 基座权重冻结为 **NF4**（4-bit NormalFloat，信息论最优的 4-bit 数据类型）+ 双重量化（量化常数再量化）。
- 配套 **分页优化器**（paged optimizer）把 optimizer states 分页到 CPU，避免长序列微调的显存尖峰。
- 效果：在 48GB 单卡上微调 65B 模型，达到全参微调 16-bit 的 99.3% 效果。
- 推理合并：`merge_and_unload()` 把 BA 加回 W₀，得到普通模型，推理无任何额外开销；也可保留 adapter 动态切换。

**变体**：
| 变体 | 机制 | 特点 |
|---|---|---|
| LoRA | ΔW = BA 低秩 | 最通用 |
| QLoRA | NF4 基座 + LoRA | 单卡微调大模型 |
| AdaLoRA | 自适应剪枝秩 | 给更重要的矩阵更多秩 |
| DoRA | 权重分解为方向+幅度 | 更接近全参微调效果 |
| PiSSA / LoRA-GA | 用 SVD/梯度信息初始化 A、B | 收敛更快 |
| LoRA+ | B 用比 A 大 16 倍的学习率 | 小改动加速收敛 |
| MoLE / LoRA 融合 | 多 adapter 按混合系数叠加 | 多任务切换 |

### 3. RLHF（Reinforcement Learning from Human Feedback）

经典三阶段流水线（InstructGPT / ChatGPT 路线）：

```
① 收集偏好数据 (prompt, 回答A, 回答B, 人标 A>B)
        ▼
② 训练奖励模型 RM：r_θ(x, y) 标量打分
   损失用 Bradley-Terry 模型：
   L_RM = -log σ(r_θ(x, y_w) - r_θ(x, y_l))
        ▼
③ PPO 强化学习：最大化奖励 + KL 约束
   max E[ r_θ(x,y) ] - β·KL(π_φ ‖ π_ref)
   防止模型为了刷奖励而"hack"奖励模型、跑偏成乱码（reward hacking）
```

**PPO 的核心机制**（GRPO 的对照组）：
- 需要 4 个模型同时在显存里：**Actor（策略）、Critic（价值函数）、Reward Model、Ref Model（参考策略）**——这就是 PPO 贵的原因。
- 重要性采样比率 `r_t(φ) = π_φ(a_t|s_t) / π_φ_old(a_t|s_t)`，裁剪到 `[1-ε, 1+ε]` 内做保守更新。
- GAE（Generalized Advantage Estimation）平衡偏差与方差。

**RLHF 的问题**：流水线长、四个模型显存爆炸、PPO 超参敏感、奖励模型容易被 hack。这直接催生了下面所有"免 RL"或直接简化的路线。

### 4. DPO（Direct Preference Optimization，直接偏好优化）

**核心洞察**：KL 约束下的 RLHF 最优解有闭式形式，代入后可以把整个 RL 问题改写成**纯粹的有监督损失**——不需要奖励模型、不需要 PPO、不需要采样：

```
L_DPO = -E_(x,yw,yl) log σ( β·log[π_θ(yw|x)/π_ref(yw|x)]  -  β·log[π_θ(yl|x)/π_ref(yl|x)] )
```

直觉：**把"好回答"相对"参考模型"的概率拉高，把"坏回答"的概率压低**，用 sigmoid 变成二分类。`β` 控制离参考模型多远（越小越自由）。

**工程要点**：
- 只需要两份权重：`policy` + `frozen ref`（ref 可以只是前向缓存 logprob），显存比 PPO 省一半以上。
- 数据就是三元组 `(prompt, chosen, rejected)`，和 RM 训练数据同构。
- 常见改良：**IPO**（去 sigmoid 防过拟合）、**SimPO**（连 ref 模型都不要，用平均 logprob + 长度归一）、**ORPO**（SFT 与偏好优化一步完成，无需 ref）。

### 5. KTO（Kahneman-Tversky Optimization）

**动机**：偏好对（A/B 对比）标注很贵。现实里更常见的是**单条二元信号**——用户点赞/点踩、留存/关闭。KTO 把这种"好/坏"的弱信号直接用起来。

**原理**：基于**前景理论（Prospect Theory）**——人对损失的厌恶约为对收益的喜爱 λ 倍（λ ≈ 2.25）：

```
价值函数 v(r) 是单调非递减的凹函数，损失区斜率 = λ × 收益区斜率
L_KTO = E[ λ_y - v(r_θ(x,y)) ]，其中 r 是相对参考模型的 log 比值
```

**与 DPO 的关系**：DPO 学的是**成对相对比较**（pairwise），KTO 学的是**逐条绝对好坏**（pointwise）；KTO 在只有二元反馈的场景与 DPO 相当甚至更好，且对数据不平衡更鲁棒。三者对比：

| | 数据需求 | 是否需要 ref 模型 | 理论依据 |
|---|---|---|---|
| DPO | (x, 好, 坏) 偏好对 | 需要 | RLHF 闭式解 |
| KTO | (x, 好) 或 (x, 坏) 单条 | 需要 | 前景理论 |
| SimPO | 偏好对 | 不需要 | DPO 去 ref 化 |

### 6. GRPO（Group Relative Policy Optimization，组相对策略优化）

**出处**：DeepSeekMath（2024.2）提出，因 DeepSeek-R1 而广为人知。**它砍掉了 PPO 的 Critic**。

**原理**：对同一个 prompt，**采样一组 G 个回答**（G 常取 8~64），用组内奖励的**相对排名**代替价值函数估计优势：

```
A_i = (R_i - mean(R_1..R_G)) / std(R_1..R_G)      ← 组内标准化即 advantage
L_GRPO = -E[ min(ratio·A, clip(ratio, 1±ε)·A) ] - β·D_KL(π_θ ‖ π_ref)
```

**为什么这样可行**：同组内 G 个回答共享同一 prompt，它们的奖励均值就是该 prompt 下"价值"的蒙特卡洛估计——**用组平均代替 Critic**，省去一个与策略同等大小的模型。

**GRPO 的两个时代**：
1. **结果监督版（Outcome reward）**：整段回答对/错一个奖励（数学答案对不对）。
2. **RLVR（RL with Verifiable Rewards）**：用**可程序化验证**的奖励替代人类偏好——数学对答案、代码过单测、逻辑过定理证明器。 reward hacking 大幅减少，这是 R1 能靠纯 RL 练出长 CoT 的关键。

**关键技巧（DeepSeek-R1 公开配方）**：
- **KL 先紧后松**：训练早期用较大 β 防止格式崩溃，后期放开让推理模式涌现。
- 纯 GRPO 直接训会导致语言混杂、可读性差 → R1 采用"冷启动 SFT（少量长 CoT 数据）→ RL → 拒绝采样再 SFT → 再 RL"的多阶段。
- 变体：**RLOO**（用 leave-one-out 组平均）、**Dr. GRPO**（去 std 归一与长度偏置）、**REINFORCE++**（去 KL 项的简化版）。

### 7. 一张表总结

| 方法 | 数据 | 需要额外模型 | 显存 | 解决什么 |
|---|---|---|---|---|
| SFT | 指令-回答对 | 无 | 低 | 学会听指令 |
| LoRA/QLoRA | 同上 | 无 | 极低（4bit） | 微调太贵 |
| RLHF+PPO | 偏好对 | RM+Critic+Ref | 极高 | 对齐人类偏好 |
| DPO | 偏好对 | Ref | 中 | PPO 太贵太脆 |
| KTO | 单条好坏 | Ref | 中 | 偏好对太贵 |
| GRPO/RLVR | prompt+可验证奖励 | Ref | 中高（免 Critic） | 推理能力涌现 |

## 第二部分：主要论文

### 【核心论文】

1. **LoRA: Low-Rank Adaptation of Large Language Models**（Hu et al., 2021）——低秩适配开山作，证明 r=16 追平全参微调。
2. **QLoRA: Efficient Finetuning of Quantized LLMs**（Dettmers et al., 2023）——NF4 + 分页优化器，单卡 48G 微调 65B。
3. **Training language models to follow instructions with human feedback（InstructGPT）**（Ouyang et al., OpenAI, 2022）——SFT+RM+PPO 三阶段范式确立。
4. **Direct Preference Optimization: Your Language Model is Secretly a Reward Model**（Rafailov et al., 2023）——免 RL 的偏好优化闭式解。
5. **KTO: Model Alignment as Prospect Theoretic Optimization**（Ethayarajh et al., 2024）——前景理论对齐，单条二元信号即可。
6. **DeepSeekMath: Pushing the Limits of Mathematical Reasoning**（Shao et al., 2024.2）——提出 GRPO。
7. **DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via RL**（DeepSeek, 2025.1）——纯 RLVR 涌现长链推理，开源复现的事实标准。
8. **LIMA: Less Is More for Alignment**（Zhou et al., 2023）——数据质量至上，1000 条精选数据足够。
9. **Secrets of RLHF in Large Language Models Part I: PPO**（2023）/ **Part II: Reward Model**（2024）——中文社区理解 RLHF 工程细节的必读。
10. **Scaling Laws for Reward Model Overoptimization**（Gao et al., 2023）——"奖励越好模型越差"的 Goldilocks 区间，理解 reward hacking 的定量依据。

### 【应用领域：领域 + 主要创新】

1. **数学推理 + 组相对优势免 Critic**：DeepSeekMath / DeepSeek-R1 / Qwen2.5-Math——GRPO+RLVR 路线把开源数学能力推到 o1 水平。
2. **代码生成 + 执行反馈奖励**：DeepSeek-Coder-V2 / SWE-RL / R2E——用单测通过率做可验证奖励，RL 直接刷编程榜单。
3. **医疗领域 + QLoRA 垂直对齐**：BioBERT/PMC-LLaMA/华佗 GPT（Huatuo）系列——医疗数据稀缺下的 4bit 低成本领域适配范式。
4. **法律领域 + 低秩适配混合**：DISC-Law-Eval / LawGPT / 北京大学 ChatLaw——法规结构化抽取 +  LoRA 微调 + 知识库外挂的组合拳。
5. **多模态 + 适配器统一**：LLaVA（视觉投影层即 adapter）/ Qwen-VL——"冻结大模型只训小投影/LoRA"成为多模态对齐默认范式。
6. **指令跟随 + 长度归一免参考模型**：SimPO——移动端/边缘部署场景下连 ref 模型的显存都省掉。
7. **Agent 行为 + 过程奖励**：WebGPT / AgentQ / SkyRL——把"网页任务是否完成"作为 RLVR 奖励，PPO/GRPO 训练浏览 Agent。
8. **奖励模型 +  Bradley-Terry 到 Generalized Bradley-Terry**：当标注从二元升级为多元排序时的理论扩展（RankDPO 等）。
9. **长上下文 + 位置编码插值微调**：Position Interpolation（PI）/ NTK-aware / YaRN——"用少量长文本 SFT 把 RoPE 外推到百万 token"，微调技术用于解决架构瓶颈的范例。
10. **模型合并 + 免训练能力缝合**：Model Soups / SLERP / DARE TIES——把多个 LoRA/微调模型的权重平均缝合，免训练获得多任务能力。

## 第三部分：项目学习库（Coding 链接）

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| huggingface/peft | LoRA/QLoRA/Prompt Tuning 的实现源头，API 设计教科书 | https://github.com/huggingface/peft |
| LLaMA-Factory | 一站式微调框架：100+ 模型、SFT/DPO/KTO/PPO/GRPO 全支持，中文文档友好 | https://github.com/hiyouga/LLaMA-Factory |
| axolotl | 声明式 yaml 配置微调，社区活跃，recipe 丰富 | https://github.com/axolotl-ai-cloud/axolotl |
| unsloth | 手写 Triton kernel 把 LoRA 微调速度提升 2-5 倍，读它学显存优化 | https://github.com/unslothai/unsloth |
| huggingface/trl | DPO/KTO/PPO/GRPO/RLOO 官方实现，论文复现第一选择 | https://github.com/huggingface/trl |
| OpenRLHF | 高性能 RLHF 框架（vLLM + Ray + DeepSpeed），对标 trl 的工业级方案 | https://github.com/OpenRLHF/OpenRLHF |
| volcengine/verl | 字节开源 RL 训练框架，GRPO/RLVR 工程化最佳实践，R1 复现主力 | https://github.com/volcengine/verl |
| microsoft/DeepSpeed-Chat | 最早的 RLHF 三阶段开源实现，理解 PPO 四模型怎么摆 | https://github.com/microsoft/DeepSpeedExamples |
| Open-Reasoner-Zero / TinyZero | 极简 GRPO/RLVR 复现（最小可跑通 R1-zero 过程），入门读代码首选 | https://github.com/Open-Reasoner-Zero/Open-Reasoner-Zero |
| predibase/lorax / lora_land | 多 LoRA 服务化推理（LoRAX），理解 adapter 动态加载与合并 | https://github.com/predibase/lorax |
| lm-sys/FastChat | 微调数据管线 + 多模型 serving，理解 chat template 生态 | https://github.com/lm-sys/FastChat |

**动手路线建议**：
1. 用 `LLaMA-Factory` 在单张消费级显卡上 QLoRA 微调一个 7B 模型（半天）。
2. 读 `trl` 的 `dpo_trainer.py` 与 `grpo_trainer.py` 各 300 行核心代码。
3. 用 `TinyZero` 在 1B 小模型上亲眼观察 GRPO 训练中"顿悟时刻"（aha moment）。
