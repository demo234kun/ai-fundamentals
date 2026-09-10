# 01 论文索引：微调与强化学习

> PDF 已从 arXiv 下载至本目录（`papers/`），点击下表"本地 PDF"即可直接阅读；arXiv 列指向在线版本与源码/项目页。

| # | 论文 | 一句话 | 本地 PDF | 解读 | arXiv |
|---|---|---|---|---|
| 1 | LoRA: Low-Rank Adaptation of Large Language Models（2021） | 低秩适配开山作，r=16 追平全参微调 | [PDF](./2106.09685_LoRA.pdf) | [解读](./2106.09685_LoRA_解读.md) | [arXiv](https://arxiv.org/abs/2106.09685) |
| 2 | QLoRA: Efficient Finetuning of Quantized LLMs（2023） | NF4 + 分页优化器，单卡 48G 微调 65B | [PDF](./2305.14314_QLoRA.pdf) | [解读](./2305.14314_QLoRA_解读.md) | [arXiv](https://arxiv.org/abs/2305.14314) |
| 3 | Training language models to follow instructions with human feedback（InstructGPT, 2022） | SFT+RM+PPO 三阶段范式确立 | [PDF](./2203.02155_InstructGPT.pdf) | [解读](./2203.02155_InstructGPT_解读.md) | [arXiv](https://arxiv.org/abs/2203.02155) |
| 4 | Direct Preference Optimization（DPO, 2023） | 免 RL 的偏好优化闭式解 | [PDF](./2305.18290_DPO.pdf) | [解读](./2305.18290_DPO_解读.md) | [arXiv](https://arxiv.org/abs/2305.18290) |
| 5 | KTO: Model Alignment as Prospect Theoretic Optimization（2024） | 前景理论对齐，单条二元信号即可 | [PDF](./2402.01306_KTO.pdf) | [解读](./2402.01306_KTO_解读.md) | [arXiv](https://arxiv.org/abs/2402.01306) |
| 6 | DeepSeekMath: Pushing the Limits of Mathematical Reasoning（2024.2） | 提出 GRPO，砍掉 PPO 的 Critic | [PDF](./2402.03300_DeepSeekMath.pdf) | [解读](./2402.03300_DeepSeekMath_解读.md) | [arXiv](https://arxiv.org/abs/2402.03300) |
| 7 | DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via RL（2025.1） | 纯 RLVR 涌现长链推理 | [PDF](./2501.12948_DeepSeek-R1.pdf) | [解读](./2501.12948_DeepSeek-R1_解读.md) | [arXiv](https://arxiv.org/abs/2501.12948) |
| 8 | LIMA: Less Is More for Alignment（2023） | 约 1000 条高质量样本即可激发指令能力 | [PDF](./2305.11206_LIMA.pdf) | [解读](./2305.11206_LIMA_解读.md) | [arXiv](https://arxiv.org/abs/2305.11206) |
| 9 | Secrets of RLHF in Large Language Models Part I: PPO（2023） | 中文社区理解 RLHF 工程细节必读 | [PDF](./2307.04964_Secrets-of-RLHF-P1.pdf) | [解读](./2307.04964_Secrets-of-RLHF-P1_解读.md) | [arXiv](https://arxiv.org/abs/2307.04964) |
| 10 | Scaling Laws for Reward Model Overoptimization（2023） | reward hacking 的 Goldilocks 区间 | [PDF](./2210.10760_Reward-Overoptimization.pdf) | [解读](./2210.10760_Reward-Overoptimization_解读.md) | [arXiv](https://arxiv.org/abs/2210.10760) |

**相关在线资源（无对应单一 arXiv 论文）**：

- Secrets of RLHF Part II（奖励模型篇）：[arXiv:2401.06080](https://arxiv.org/abs/2401.06080)
- 应用领域论文（DeepSeek-Coder、SimPO、模型合并等）见 [基础知识文档](../README.md) 第二部分列表，按标题可在 arXiv 检索。
