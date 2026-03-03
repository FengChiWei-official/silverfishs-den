# KTO: Model Alignment as Prospect Theoretic Optimization

## 基本信息
- **作者**：Kawin Ethayarajh, Winnie Xu, Niklas Muennighoff, et al. (Contextual AI)
- **发表**：arXiv 2024
- **论文链接**：[https://arxiv.org/abs/2402.01306](https://arxiv.org/abs/2402.01306)

## 核心问题
DPO 虽然简化了 RLHF，但仍需要成对的偏好数据（Pairwise Preference Data, e.g., A > B）。这种数据收集成本高，且很多现有的数据只有“点赞/点踩”（Thumbs-up/Thumbs-down）的二元反馈，无法构成配对。如何直接利用这种非配对的二元反馈数据进行对齐？

## 主要贡献
1. 提出了 **KTO (Kahneman-Tversky Optimization)**：一种无需配对数据的对齐方法。
2. 基于**前景理论 (Prospect Theory)**（由 Kahneman 和 Tversky 提出），建模人类对损失和收益的非对称感知（损失厌恶）。
3. 证明了 KTO 在仅使用二元反馈数据的情况下，能达到甚至超过使用配对数据的 DPO 的效果。

## 技术要点
- **前景理论**：人类对“损失”的敏感度远高于同等大小的“收益”。
- **KTO Loss**：
  试图最大化“好样本”（$y_{desirable}$）的效用，同时最小化“坏样本”（$y_{undesirable}$）的效用。
  它不需要 $y_w$ 和 $y_l$ 同时出现，只需要知道某个样本 $y$ 是好的还是坏的。
  $$ L_{KTO} \approx \text{Weight} \cdot (\text{Value Function of Policy Ratio}) $$
- **数据需求**：只需要 $(x, y, label)$，其中 label 是 boolean (good/bad)。这使得利用历史日志数据（用户点击、点赞）变得非常容易。

## 实验结果
- 在多个基准测试中，KTO 在使用非配对数据的情况下，性能匹敌甚至优于使用配对数据的 DPO。
- 解决了数据稀缺问题，因为非配对数据远比配对数据容易获取。

## 创新点
- **引入经济学理论**：首次将行为经济学中的前景理论引入 LLM 对齐。
- **解耦数据约束**：打破了 RLHF/DPO 对配对数据的依赖。

## 缺点与局限
- 超参数（如 $\lambda_D, \lambda_U$）较多，调节起来比 DPO 稍复杂。
- 理论推导相对晦涩。

## 应用价值
- 非常适合工业界场景，因为企业通常拥有大量的用户反馈日志（点赞/点踩），但很少有完美的 A/B 测试配对数据。

## 相关笔记
- [[06-DPO-直接偏好优化]]
- [[05-PPO-近端策略优化]]

## 个人总结
KTO 是对齐技术的又一次进化。如果说 DPO 解决了“算法复杂”的问题，KTO 则解决了“数据昂贵”的问题。它让利用海量用户交互日志进行模型持续进化成为可能。
