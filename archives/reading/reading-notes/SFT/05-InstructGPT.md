# InstructGPT: Training language models to follow instructions with human feedback

## 基本信息
- **作者**：Long Ouyang, Jeff Wu, Xu Jiang, et al. (OpenAI)
- **发表**：NeurIPS 2022
- **论文链接**：[https://arxiv.org/abs/2203.02155](https://arxiv.org/abs/2203.02155)
- **博客链接**：[https://openai.com/research/instruction-following](https://openai.com/research/instruction-following)

## 核心问题
大型语言模型（如 GPT-3）虽然强大，但经常生成不真实、有毒或对用户无帮助的输出。这是因为预训练的目标（预测下一个词）与用户意图（遵循指令、安全、有用）不一致（Misalignment）。如何让模型与人类意图对齐？

## 主要贡献
1. 提出了 **RLHF (Reinforcement Learning from Human Feedback)** 的完整流程，用于微调 GPT-3。
2. 证明了 1.3B 参数的 InstructGPT 模型在人类评估中优于 175B 的 GPT-3 Base 模型。
3. 详细分析了对齐带来的好处（真实性、无害性）以及潜在的代价（对齐税）。

## 技术要点
RLHF 三阶段流程：
1. **SFT (Supervised Fine-Tuning)**：收集人类编写的演示数据（Prompt + Answer），微调 GPT-3。
2. **RM (Reward Modeling)**：收集模型生成的多个输出，由人类进行排序（Ranking），训练一个奖励模型（Reward Model）来预测人类偏好。
   - Loss: Pairwise Ranking Loss.
3. **PPO (Proximal Policy Optimization)**：使用 RM 作为奖励函数，通过 PPO 算法进一步微调 SFT 模型，使其生成能获得更高奖励的回复。
   - 目标函数包含：RM 的奖励 + KL 散度惩罚（防止模型偏离 SFT 模型太远）。

## 实验结果
- **偏好度**：人类标注员显著更喜欢 InstructGPT 的输出。
- **真实性**：在 TruthfulQA 数据集上表现更好，幻觉减少。
- **无害性**：生成的有毒内容减少。
- **泛化性**：模型能够遵循它在训练中未见过的指令类型。

## 创新点
- 将强化学习引入到 LLM 的微调中，利用人类反馈信号解决不可微的优化目标（如“有用性”、“安全性”）。
- 确立了 LLM 对齐的标准范式（SFT -> RM -> PPO）。

## 缺点与局限
- **人工标注成本高**：需要大量高质量的人类标注数据。
- **对齐税**：在某些特定任务（如代码生成、翻译）上，性能可能略有下降。
- **价值观植入**：模型的价值观取决于标注者的偏好，可能存在偏见。

## 应用价值
- ChatGPT 的前身技术，开启了对话式 AI 的新时代。
- 为后续所有对齐工作（Llama-2-Chat, Claude 等）奠定了基础。

## 相关笔记
- [[04-SFT-监督式微调]]
- [[05-PPO-近端策略优化]]
- [[08-RLHF-强化学习人类反馈]]

## 关键公式
$$ \text{loss}(\theta) = - \mathbb{E}_{(x, y_w, y_l) \sim D} [\log \sigma (r_\theta(x, y_w) - r_\theta(x, y_l))] $$
(Reward Model 的 Pairwise Loss)

## 个人总结
InstructGPT 是 LLM 发展史上的里程碑。它证明了“对齐”（Alignment）与“能力”（Capability）同样重要。通过 RLHF，OpenAI 成功地驯服了 GPT-3 这头野兽，使其变得可用、好用。
