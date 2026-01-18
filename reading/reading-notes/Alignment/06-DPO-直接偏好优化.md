# Direct Preference Optimization (DPO)

## 基本信息
- **作者**：Rafael Rafailov, Archit Sharma, Eric Mitchell, et al. (Stanford)
- **发表**：NeurIPS 2023 (Best Paper Runner-up)
- **论文链接**：[https://arxiv.org/abs/2305.18290](https://arxiv.org/abs/2305.18290)

## 核心问题
现有的 RLHF 流程（SFT -> RM -> PPO）复杂、不稳定且计算昂贵。需要训练奖励模型，然后用强化学习算法（PPO）优化策略，这涉及多个模型的加载和复杂的超参调试。能否绕过奖励模型和 PPO，直接利用人类偏好数据优化语言模型？

## 主要贡献
1. 提出了 **DPO (Direct Preference Optimization)**：一种无需显式奖励模型、无需强化学习的对齐算法。
2. 理论证明了 RLHF 的目标函数（最大化奖励 + KL 约束）等价于一个简单的分类损失函数。
3. 实验表明 DPO 在性能上媲美甚至超越 PPO，且训练速度更快、显存占用更少、超参更少。

## 技术要点
- **理论推导**：作者利用 RL 目标的最优解形式，反推出了奖励函数 $r(x,y)$ 与最优策略 $\pi^*(y|x)$ 之间的关系：
  $$ r(x,y) = \beta \log \frac{\pi^*(y|x)}{\pi_{ref}(y|x)} + Z(x) $$
- **DPO Loss**：将上述关系代入 Bradley-Terry 偏好模型，消去奖励项，得到直接针对策略 $\pi_\theta$ 的损失函数：
  $$ L_{DPO}(\pi_\theta; \pi_{ref}) = - \mathbb{E}_{(x, y_w, y_l) \sim D} [\log \sigma (\beta \log \frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)})] $$
  - 本质上是一个二分类任务：增加“好回复”相对于 Reference Model 的概率，降低“坏回复”的概率。

## 实验结果
- 在摘要生成（TL;DR）和对话任务（Anthropic HH）上，DPO 取得了与 PPO 相当或更好的胜率。
- 训练稳定性显著提高，不再受 PPO 训练崩溃的困扰。

## 创新点
- **大道至简**：将复杂的 RL 问题转化为简单的监督学习（分类）问题。
- **Your Language Model is Secretly a Reward Model**：揭示了语言模型本身就隐式地包含了一个奖励模型。

## 缺点与局限
- 仍然需要成对的偏好数据（Pairwise Data）。
- 对 Reference Model 的选择和 Batch Size 比较敏感。
- 缺乏像 PPO 那样的探索（Exploration）机制，可能受限于数据集的分布。

## 应用价值
- 迅速成为开源社区微调对齐的首选算法（如 Zephyr, Tulu 2 等模型均采用 DPO）。
- 大幅降低了 LLM 对齐的门槛和成本。

## 相关笔记
- [[05-PPO-近端策略优化]]
- [[07-KTO-卡尼曼特维斯基优化]]
- [[08-RLHF-强化学习人类反馈]]

## 个人总结
DPO 是 LLM 对齐领域的“LoRA 时刻”。它极大地简化了流程，让对齐变得触手可及。它的出现标志着我们对 LLM 偏好学习的理解进入了一个新的阶段——从“黑盒 RL”转向“白盒优化”。
