# Proximal Policy Optimization (PPO)

## 基本信息
- **作者**：John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, Oleg Klimov (OpenAI)
- **发表**：arXiv 2017
- **论文链接**：[https://arxiv.org/abs/1707.06347](https://arxiv.org/abs/1707.06347)

## 核心问题
在强化学习（RL）中，策略梯度（Policy Gradient）方法通常对步长（Step Size）非常敏感。步长太小，收敛慢；步长太大，策略更新过猛导致性能崩溃且难以恢复。如何设计一种既稳定又高效的策略优化算法？

## 主要贡献
1. 提出了 **PPO (Proximal Policy Optimization)** 算法，作为 TRPO (Trust Region Policy Optimization) 的简化和改进版。
2. 引入了 **Clipped Surrogate Objective**（截断代理目标），限制了新旧策略之间的差异，防止策略更新幅度过大。
3. 证明了 PPO 在多种连续控制任务和 Atari 游戏中表现优异，且实现简单、样本效率高。

## 技术要点
- **Policy Ratio**: $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$，表示新旧策略的概率比。
- **Clipped Objective**:
  $$ L^{CLIP}(\theta) = \hat{\mathbb{E}}_t [\min(r_t(\theta)\hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)\hat{A}_t)] $$
  - $\hat{A}_t$ 是优势函数（Advantage）。
  - $\epsilon$ 是超参数（通常为 0.1 或 0.2）。
  - 该公式确保了当 $r_t(\theta)$ 偏离 1 太多时，梯度更新会被截断，从而保证策略更新在“信任域”内。

## 实验结果
- 在 MuJoCo 物理模拟和 Atari 游戏中，PPO 均取得了 SOTA 或接近 SOTA 的成绩。
- 相比 TRPO，PPO 更容易实现（不需要计算二阶导数/Hessian 矩阵），计算成本更低。

## 创新点
- 用简单的 Clip 操作替代了 TRPO 中复杂的 KL 散度约束，达到了类似的效果（限制策略更新步幅）。

## 缺点与局限
- 仍然对超参数（如 clip range, entropy coefficient）敏感。
- 在 LLM 微调中，PPO 的训练过程极其不稳定，显存占用大（需要加载 Policy Model, Reference Model, Reward Model, Value Model）。

## 应用价值
- **RLHF 的核心算法**：PPO 是目前 InstructGPT, Llama-2 等模型进行 RLHF 阶段微调的标准算法。
- 广泛应用于机器人控制、游戏 AI 等领域。

## 相关笔记
- [[08-RLHF-强化学习人类反馈]]
- [[06-DPO-直接偏好优化]]
- [[05-InstructGPT]]

## 个人总结
PPO 是深度强化学习领域的经典之作。在 LLM 时代，它焕发了第二春，成为连接人类意图与模型行为的桥梁。虽然 DPO 等新方法正在挑战它的地位，但 PPO 仍然是理解 RLHF 不可绕过的基石。
