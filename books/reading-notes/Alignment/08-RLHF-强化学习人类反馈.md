# RLHF: Deep Reinforcement Learning from Human Preferences

## 基本信息
- **作者**：Paul F. Christiano, Jan Leike, Tom B. Brown, et al. (OpenAI & DeepMind)
- **发表**：NIPS 2017
- **论文链接**：[https://arxiv.org/abs/1706.03741](https://arxiv.org/abs/1706.03741)
- **后续重要论文**：Learning to summarize from human feedback (Stiennon et al., 2020)

## 核心问题
在很多复杂的任务中（如“后空翻”、“写一篇好摘要”），很难定义一个精确的奖励函数（Reward Function）。如果我们无法写出奖励函数，如何利用强化学习来训练智能体？

## 主要贡献
1. 提出了 **RLHF (Reinforcement Learning from Human Feedback)** 的基础框架：通过学习一个奖励模型（Reward Model）来代理人类的偏好。
2. 展示了仅通过少量的非专家人类反馈（比较两个视频片段哪个更好），就能训练智能体完成复杂的控制任务（如 MuJoCo 机器人后空翻）和 Atari 游戏。
3. 证明了这种方法可以扩展到大规模神经网络和复杂环境中。

## 技术要点
- **偏好收集**：向人类展示两个轨迹片段 $\sigma^1, \sigma^2$，让人类判断哪个更好。
- **奖励建模**：训练一个神经网络 $\hat{r}$ 来预测偏好概率：
  $$ P[\sigma^1 \succ \sigma^2] = \frac{\exp \sum \hat{r}(o_t^1)}{\exp \sum \hat{r}(o_t^1) + \exp \sum \hat{r}(o_t^2)} $$
- **策略优化**：使用学习到的 $\hat{r}$ 作为奖励函数，运行标准的 RL 算法（如 PPO/A2C）来优化策略。

## 实验结果
- 在 MuJoCo 环境中，智能体学会了后空翻等复杂动作，而这些动作很难通过手工编写奖励函数来描述。
- 在 Atari 游戏中，智能体有时能发现人类未察觉的策略。

## 创新点
- **Human-in-the-loop RL**：将人类反馈显式地纳入 RL 循环，解决了 Reward Engineering 的难题。
- **可扩展性**：不需要人类实时指导，而是通过训练 Reward Model 来泛化人类的评价标准。

## 缺点与局限
- **反馈效率**：虽然比实时指导高效，但仍需要大量的人类比较数据。
- **Reward Hacking**：智能体可能会利用 Reward Model 的漏洞刷分，导致行为不符合人类真实意图。

## 应用价值
- 这是 ChatGPT 背后的核心思想起源。虽然当时主要用于控制任务，但其思想直接启发了后来在 NLP 领域的应用（InstructGPT）。

## 相关笔记
- [[05-InstructGPT]]
- [[05-PPO-近端策略优化]]
- [[06-DPO-直接偏好优化]]

## 个人总结
这篇论文是 RLHF 的开山之作（之一）。它从根本上改变了我们定义“目标”的方式：从“写代码定义目标”转变为“通过示例展示目标”。这种范式的转变是通向通用人工智能（AGI）的关键一步，因为人类的价值观很难被硬编码。
