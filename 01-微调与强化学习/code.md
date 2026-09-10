# 01 代码学习库：微调与强化学习

> 按"先跑通 → 再读源码"的顺序排列。克隆后建议先看各自 quickstart 文档。

| 仓库 | 适合学什么 | 链接 |
|---|---|---|
| LLaMA-Factory | 一站式微调框架：100+ 模型、SFT/DPO/KTO/PPO/GRPO 全支持，中文文档友好 | https://github.com/hiyouga/LLaMA-Factory |
| huggingface/peft | LoRA/QLoRA/Prompt Tuning 的实现源头，API 设计教科书 | https://github.com/huggingface/peft |
| huggingface/trl | DPO/KTO/PPO/GRPO/RLOO 官方实现，论文复现第一选择 | https://github.com/huggingface/trl |
| unsloth | 手写 Triton kernel 把 LoRA 微调提速 2-5 倍，读它学显存优化 | https://github.com/unslothai/unsloth |
| axolotl | 声明式 yaml 配置微调，社区活跃，recipe 丰富 | https://github.com/axolotl-ai-cloud/axolotl |
| OpenRLHF | 高性能 RLHF 框架（vLLM + Ray + DeepSpeed），工业级方案 | https://github.com/OpenRLHF/OpenRLHF |
| volcengine/verl | 字节开源 RL 训练框架，GRPO/RLVR 工程化最佳实践，R1 复现主力 | https://github.com/volcengine/verl |
| microsoft/DeepSpeed-Chat | 最早的 RLHF 三阶段开源实现，理解 PPO 四模型怎么摆 | https://github.com/microsoft/DeepSpeedExamples |
| Open-Reasoner-Zero | 极简 GRPO/RLVR 复现（最小可跑通 R1-zero 过程），入门读代码首选 | https://github.com/Open-Reasoner-Zero/Open-Reasoner-Zero |
| TinyZero | 1B 小模型上的 GRPO 最小复现，观察"顿悟时刻" | https://github.com/Jiayi-Pan/TinyZero |
| predibase/lorax | 多 LoRA 服务化推理（LoRAX），理解 adapter 动态加载与合并 | https://github.com/predibase/lorax |
| lm-sys/FastChat | 微调数据管线 + 多模型 serving，理解 chat template 生态 | https://github.com/lm-sys/FastChat |

**动手路线**：
1. 用 `LLaMA-Factory` 在单张消费级显卡上 QLoRA 微调一个 7B 模型（半天）。
2. 读 `trl` 的 `dpo_trainer.py` 与 `grpo_trainer.py` 各 300 行核心代码。
3. 用 `TinyZero` 在 1B 小模型上亲眼观察 GRPO 训练中的"顿悟时刻"（aha moment）。
