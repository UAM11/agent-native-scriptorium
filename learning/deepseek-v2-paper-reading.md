# DeepSeek-V2 论文阅读笔记

- Created: 2026-04-30
- Updated: 2026-04-30
- Type: learning
- Status: draft
- Tags: deepseek, deepseek-v2, mla, moe, long-context, alignment, paper-reading
- Model: GPT-5.4
- Harness: Codex
- Source: first-pass close reading of the official DeepSeek-V2 paper, rewritten as the system-integration half of DeepSeek stage 2

## 背景

这篇笔记对应 DeepSeek 路线第二阶段里偏“系统整合”的一半。  
如果 `DeepSeekMoE` 主要回答的是：

> FFN 这一侧的稀疏架构怎样改得更合理？

那么 `DeepSeek-V2` 真正回答的是：

> 怎样把 `DeepSeekMoE` 和 `MLA` 组合起来，做出一代强、经济、长上下文、还能真正部署的模型？

## 论文信息

- 标题：[DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model](https://arxiv.org/abs/2405.04434)
- arXiv：`2405.04434`
- 首版日期：`2024-05-07`
- 常被引用的版本：`v5`, `2024-06-19`
- 在 DeepSeek 路线中的位置：`DeepSeekMoE + MLA`

## 原文目录地图

主干结构是：

1. `Introduction`
2. `Architecture`
   - `2.1 Multi-Head Latent Attention`
   - `2.2 DeepSeekMoE`
3. `Pre-Training`
4. `Alignment`
5. `Conclusion, Limitation, and Future Work`

值得额外看的附录：

- `Appendix B`: `DeepSeek-V2-Lite`
- `Appendix D`: attention ablations，尤其是 `MHA / GQA / MQA / MLA` 的对比

## Abstract

摘要已经把这代模型的成果压得很清楚：

- `236B` total parameters
- `21B` activated parameters per token
- `128K` context
- `8.1T` pretraining tokens
- `SFT + RL`

最重要的是它把第二阶段的两个关键词明确写在一起：

- `DeepSeekMoE`
- `MLA`

这意味着：

> `V2` 不是只有一个创新点，  
> 它是 FFN 稀疏化和注意力压缩第一次被完整合体。

## 1. Introduction

Introduction 的逻辑很直：

1. 模型越大，能力通常越强。
2. 但训练成本和推理吞吐会快速恶化。
3. 所以问题不再只是“要不要继续放大”，而是：

> 怎样在保持强能力的同时，把训练成本、KV cache 和推理吞吐一起优化？

这一节给出的答案分三块：

1. 注意力侧：`MLA`
2. FFN 侧：`DeepSeekMoE`
3. 系统侧：通信与训练优化

所以从一开始，`V2` 就不是“单点提分”，而是“整体经济性重构”。

## 2. Architecture

这一章是整篇 `V2` 的中心。

### 2.1 Multi-Head Latent Attention

这一节要解决的是推理时最现实的瓶颈之一：

> `MHA` 的 KV cache 太重了。

论文先把已有替代路线摆出来：

- `MQA`
- `GQA`

然后指出它们的问题：

- 它们虽然省 cache
- 但会损失能力

于是 `MLA` 的目标不是只压缩，而是：

> 用 low-rank key-value joint compression，在尽量保住甚至超过 `MHA` 能力的前提下，把 KV cache 大幅压小。

这一节最值得看的表是 `Table 1`。  
它把不同注意力机制的 KV cache 规模放在一起比较，并给出一个很强的结论：

- `MLA` 的 KV cache 接近非常小的分组水平
- 但能力目标不是退到 `MQA/GQA` 的妥协线，而是保持 strong 甚至 stronger

这就是第二阶段注意力线的本质：

> 不是简单“少存一点 KV”，而是“让长上下文和高吞吐真正可用”。

### 2.2 DeepSeekMoE

这一节在 `V2` 里不是重复 `DeepSeekMoE` 论文，而是把那套架构放进真实大模型训练系统里。

除了 basic architecture 外，它额外强调了几件事：

- `device-limited routing`
- `auxiliary loss for load balance`
- `token-dropping strategy`

也就是说，`V2` 里的 `DeepSeekMoE` 已经不是“概念版 MoE”，而是“能在真实集群上跑的大模型版 MoE”。

## 3. Pre-Training

这一章把第二阶段真正拉到了模型级别。

最重要的事实是：

- `V2` 预训练用了 `8.1T` tokens
- 模型支持 `128K` context
- 长上下文扩展不是原生从头训到 `128K`，而是先预训练，再用 `YaRN` 做扩展

### 3.1 Experimental Setups / Data Construction

作者沿用了 `DeepSeek 67B` 的数据处理阶段，但：

- 扩大了数据量
- 提升了数据质量
- 增加了中文数据
- 讨论了 pre-training data debiasing

这说明第二阶段虽然换了架构，但并没有把第一阶段对数据工程的重视扔掉。

### 3.1.4 Long Context Extension

这节很值得单独记住。

它说明：

- `V2` 的默认 context 先从 `4K` 起步
- 再借助 `YaRN` 扩展到 `128K`
- 实际只额外训练了 `1000` steps、`32K` sequence length
- 但模型在 `128K` 的 NIAH 测试里仍然表现稳定

这部分对应第二阶段一个非常现实的结论：

> 长上下文不只是模型结构问题，也是 curriculum 和 extension strategy 问题。

### 3.2 Training and Inference Efficiency

这是 `V2` 最不能跳过的一节。

作者直接给出第二阶段最硬的工程结果：

- 相比 `DeepSeek 67B`，训练成本降低 `42.5%`
- 实际部署下 KV cache 降低 `93.3%`
- 单节点 `8 x H800` 上最大生成吞吐提升到 `5.76x`

这几组数字之所以重要，是因为它们把“经济、强、快”这三个词真正落到了可比较指标上。

## 4. Alignment

和第一阶段相比，这里最值得注意的变化不是有没有 `SFT`，而是：

- `V2` 已经明确把 `RL` 写进正式 pipeline
- 这说明 DeepSeek 的后训练路线在第二阶段已经开始发生变化

### 4.1 Supervised Fine-Tuning

作者给出的核心设定包括：

- `1.5M` instruction tuning instances
- 其中 `1.2M` helpfulness，`0.3M` safety
- `2` epochs
- `5e-6` learning rate

这表明第二阶段并没有抛弃第一阶段的数据化、稳妥式 `SFT`，而是在更强 base 之上继续做高质量 instruction tuning。

### 4.2 Reinforcement Learning

这里有两个层次值得记：

1. 方法层：第二阶段已经开始把 `RL` 视为标准后训练组成部分。
2. 工程层：作者专门写了 `RL` 训练效率优化，包括：
   - train/inference different parallel strategies 的 hybrid engine
   - 用 `vLLM` 做大 batch inference backend
   - CPU/GPU offload scheduling

这一节很关键，因为它预告了后面 `R1` 那条线并不是凭空出现的。

### 4.4 Discussion

`V2` 的 discussion 很有价值，至少有两条：

- `SFT` 数据量不能想当然地无限压缩，足够的数据量仍然重要
- `RL` 会带来 alignment tax，需要在开放式对话质量与标准 benchmark 之间做平衡

这说明第二阶段对齐线的主题已经不再是“要不要 RL”，而是“怎么让 RL 带来收益，同时控制代价”。

## 5. Conclusion, Limitation, and Future Work

到 conclusion，这一代模型的定位已经很明确：

- 架构上：`DeepSeekMoE + MLA`
- 训练上：强 base + 经济训练
- 推理上：极大压缩 KV cache、提升吞吐
- 对齐上：`SFT + RL`

它的价值不只是单模型成绩，而是把第二阶段的整体方向钉死了：

> 继续做大模型，不必默认等于更贵、更慢、更难部署。

## 读完后最该带走的 7 个点

1. `V2` 把第二阶段真正定义成“经济性重构”，而不是单点提分。
2. `MLA` 是第二阶段注意力线的核心，因为它直面 KV cache 瓶颈。
3. `DeepSeekMoE` 在 `V2` 里从架构想法进化成了真实训练系统的一部分。
4. `8.1T` tokens 和 `128K` context 说明第二阶段已经进入更大规模预训练和长上下文阶段。
5. `YaRN` 扩展表明长上下文不一定全靠从头原生训练，也可以通过扩展策略完成。
6. `SFT + RL` 说明后训练路线已经从第一阶段的 `SFT + DPO` 往后移动。
7. 训练成本、KV cache 和吞吐这三组工程指标，是理解 `V2` 最不能跳过的部分。

## 如何回到路线笔记

回到 [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md) 时，可以只记一句：

> 第二阶段在注意力和整合这一侧的答案，是 `DeepSeekMoE + MLA`。

## 相关笔记

- [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md)
- [DeepSeekMoE 论文阅读笔记](deepseek-moe-paper-reading.md)
