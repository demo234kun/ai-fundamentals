# 06 代码学习库：自动进化与自我改进

| 仓库/资源 | 适合学什么 | 链接 |
|---|---|---|
| lobehub/awesome-rsi | RSI 研究地图：模型/权重/harness/具身/评测/安全全谱系分类目录（本模块论文谱系主要来源） | https://github.com/lobehub/awesome-rsi |
| self-developing-agents.github.io | 三件套项目页：三基准的交互数据与轨迹可视化 | https://self-developing-agents.github.io/ |
| PostTrainBench | 自动化后训练基准：一块 H100 十小时自主训练（Aspire 的显式任务前身） | 见 awesome-rsi 索引（arXiv 2026） |
| laude-institute/terminal-bench | HarnessDev 的评测环境之一（89 题终端任务） | https://github.com/laude-institute/terminal-bench |
| HKUDS/LightRAG / run-llama/llama_index | 自建 harness 实验的检索/记忆底座 | https://github.com/HKUDS/LightRAG |
| MineDojo/Voyager | 技能沉淀闭环的最简可复现版本（先跑这个再谈进化） | https://github.com/MineDojo/Voyager |
| OpenPipe/ART | 用可验证奖励在真实 harness 轨迹上训练（经验→参数路线） | https://github.com/OpenPipe/ART |
| All-Hands-AI/OpenHands | harness 作为被评测工件的对照平台 | https://github.com/All-Hands-AI/OpenHands |

**动手路线**：
1. 读 awesome-rsi，按"权重级 / harness 级 / 技能级"给自己画一张演化谱系图。
2. 复现 Voyager 技能入库循环，体会"执行成功才入库"的留存验证思想。
3. 用 terminal-bench 跑一个固定 harness，再手动迭代 3 版（模仿 HarnessDev 的反馈集/held-out 分离），观察可见分与 held-out 分的相关性。
4. 对照三件套的三个失败模式（目标错 / 经验不迁移 / 机制不存活），给自己的 agent 系统做一次审计。
