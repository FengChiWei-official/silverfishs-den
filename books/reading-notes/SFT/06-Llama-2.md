# Llama 2: Open Foundation and Fine-Tuned Chat Models

## 基本信息
- **作者**：Hugo Touvron, Louis Martin, Kevin Stone, et al. (Meta AI)
- **发表**：arXiv 2023
- **论文链接**：[https://arxiv.org/abs/2307.09288](https://arxiv.org/abs/2307.09288)
- **模型链接**：[https://huggingface.co/meta-llama](https://huggingface.co/meta-llama)

## 核心问题
如何构建一个开源的、可商用的、且在安全性和有用性上能与闭源模型（如 ChatGPT）媲美的大语言模型？

## 主要贡献
1. 发布了 **Llama 2** 系列模型（7B, 13B, 70B），包括预训练基座（Base）和对话微调版本（Chat）。
2. 详细公开了从预训练到 RLHF 对齐的完整技术细节，包括数据处理、训练超参、安全评估等。
3. 提出了 **GQA (Grouped-Query Attention)** 和 **Ghost Attention (GAtt)** 等新技术。

## 技术要点
- **预训练**：
  - 2T Tokens 的训练数据（比 Llama 1 多 40%）。
  - 上下文长度扩展到 4096。
  - 70B 模型使用了 GQA 以加速推理。
- **SFT**：
  - 使用了约 27,000 条高质量的 SFT 数据（强调质量优于数量）。
- **RLHF**：
  - 收集了超过 100万条人类偏好比较数据。
  - 使用了两个独立的 Reward Model（Helpfulness RM 和 Safety RM）。
  - 采用了 **Rejection Sampling** (拒绝采样) 和 **PPO** 两种算法进行优化。
- **Ghost Attention (GAtt)**：
  - 解决多轮对话中 System Prompt 被遗忘的问题，通过在多轮对话的所有 User Message 前拼接 System Prompt 进行训练。

## 实验结果
- Llama 2-Chat 在众多基准测试中超越了所有同量级的开源模型。
- 在人类评估中，Llama 2-Chat 70B 与 ChatGPT (GPT-3.5) 互有胜负。
- 安全性显著提升，拒绝回答有害指令的能力增强。

## 创新点
- **开源精神**：不仅开源权重，还详细披露了“配方”，极大地推动了开源社区的发展。
- **安全性重视**：将安全性提升到与有用性同等的高度，并在 RLHF 阶段专门优化。
- **Rejection Sampling**：在 RLHF 早期阶段使用拒绝采样作为一种简单有效的对齐手段。

## 缺点与局限
- 相比 GPT-4，代码能力和逻辑推理能力仍有差距。
- 过于“安全”：有时会过度拒绝无害的请求（False Refusal）。

## 应用价值
- 目前最流行的开源 LLM 基座之一，被广泛用于各种垂直领域的微调和应用开发。
- 确立了开源大模型的标杆。

## 相关笔记
- [[05-InstructGPT]]
- [[04-SFT-监督式微调]]
- [[06-DPO-直接偏好优化]]

## 个人总结
Llama 2 的发布是开源 LLM 的转折点。它打破了闭源模型的技术垄断，让全世界的研究者和开发者都能接触到最先进的 LLM 技术。其技术报告是一份非常详尽的 LLM 训练教科书。
