# DeepSeek 第二阶段论文阅读笔记：DeepSeekMoE 与 DeepSeek-V2

- Created: 2026-04-30
- Updated: 2026-04-30
- Type: learning
- Status: draft
- Tags: deepseek, deepseekmoe, deepseek-v2, moe, mla, long-context, paper-reading
- Model: GPT-5.4
- Harness: Codex
- Source: first-pass close reading of the official DeepSeekMoE and DeepSeek-V2 papers, organized as the second stage of the DeepSeek research route

## 背景

这篇笔记对应 DeepSeek 路线里的第二阶段。  
如果说第一阶段 `DeepSeek LLM / V1` 主要回答的是：

> dense 基础模型到底该怎么训练、怎么扩展才更合理？

那么第二阶段真正开始回答的是：

> 在这套训练规律已经相对清楚之后，怎样把“继续做大模型”这件事做得更便宜、更快、也更适合真实部署？

## 这篇笔记怎么用

建议把这篇笔记当成第二阶段的专题入口，而不是一次性读完的终稿。

阅读顺序建议是：

1. 先看“为什么第二阶段要一起读两篇”
2. 再看 `DeepSeekMoE`，理解 FFN 这一侧为什么会走向更细粒度、更强专门化的 MoE
3. 再看 `DeepSeek-V2`，理解它如何把 `DeepSeekMoE + MLA` 真正组合成一代可训练、可部署的模型
4. 最后回到路线笔记，看第二阶段在整条主线里的位置

## 为什么第二阶段要一起读两篇

第二阶段最好不要只看 `DeepSeek-V2`，也不要只看 `DeepSeekMoE`。

因为这两篇论文解决的是同一个阶段里的两半问题：

- `DeepSeekMoE`：重点在 FFN 侧，回答“MoE 为什么要这样改，才能让专家更专门化、更不冗余”
- `DeepSeek-V2`：重点在系统整合，回答“怎样把 `DeepSeekMoE` 和 `MLA` 组合成一代经济、强大、长上下文、可对齐的模型”

简单说：

> `DeepSeekMoE` 更像“架构发明”，  
> `DeepSeek-V2` 更像“架构发明 + 大规模工程落地”。

## Part I. DeepSeekMoE

### 论文信息

- 标题：[DeepSeekMoE: Towards Ultimate Expert Specialization in Mixture-of-Experts Language Models](https://arxiv.org/abs/2401.06066)
- arXiv：`2401.06066`
- 首版日期：`2024-01-11`
- 它在路线里的位置：`FFN -> DeepSeekMoE`

### 原文目录地图

论文的主干结构是：

1. `Introduction`
2. `Preliminaries: Mixture-of-Experts for Transformers`
3. `DeepSeekMoE Architecture`
   - `3.1 Fine-Grained Expert Segmentation`
   - `3.2 Shared Expert Isolation`
   - `3.3 Load Balance Consideration`
4. `Validation Experiments`
5. `Scaling up to DeepSeekMoE 16B`
6. `Alignment for DeepSeekMoE 16B`
7. `DeepSeekMoE 145B Ongoing`
8. `Related Work`
9. `Conclusion`

### Abstract：这篇论文到底在讲什么

摘要的任务非常集中：

- 先说传统 `MoE` 在“专家专门化”上并没有做到理想状态
- 再说 `DeepSeekMoE` 的两条核心策略：
  - finer-grained expert segmentation
  - shared expert isolation
- 最后给出几组结果，证明它能在更少计算下接近 dense upper bound，或者打赢更传统的 GShard 式 MoE

从摘要就能抓住这篇论文的本质：

> 它不是在说 “MoE 能省算力” 这种老结论，  
> 它是在说 “怎样把 MoE 做到更强的 expert specialization”。

### 1. Introduction：为什么已有 MoE 还不够

这一节的逻辑很清楚：

1. 大模型继续放大时，训练成本越来越高，所以 MoE 很自然会被拿来做稀疏扩展。
2. 但现有 MoE 做法虽然省算，却未必真的让专家变得“各司其职”。
3. 作者认为传统 MoE 的核心问题不是只有成本，而是两类知识层面的缺陷：
   - `knowledge hybridity`：一个专家被迫学太多彼此差异很大的知识
   - `knowledge redundancy`：不同专家重复学习公共知识
4. 因此，论文的目标不是简单复制 GShard，而是追求更高水平的 expert specialization。

这节最值得记住的一句话是：

> 第二阶段不是从“参数更多”出发，而是从“专家有没有真正分工”出发。

### 2. Preliminaries：先把传统 Transformer-MoE 写清楚

这节做的是铺垫，不是创新本身。

作者先把标准 Transformer 里的 FFN 替换成 MoE layer 的常规写法整理清楚：

- 每个 expert 结构上就是一个 FFN
- router 为每个 token 选择 top-k experts
- 稀疏门控保证每个 token 只激活少量专家

这节的作用是：

- 告诉你 `DeepSeekMoE` 不是另起炉灶
- 它是在“传统 top-k routed MoE”之上做结构性修正

### 3. DeepSeekMoE Architecture：第二阶段真正的架构核心

这节是整篇 `DeepSeekMoE` 的核心。

#### 3.1 Fine-Grained Expert Segmentation

作者的思路是：

- 如果 expert 数量太少，每个 expert 就容易被迫承载过多异质知识
- 所以不如把同样的专家参数切得更细
- 同时把激活的 expert 数也按比例增大

论文里的关键约束是：

- 总 expert 参数量不变
- 计算量不变
- 但 expert 粒度更细、组合空间更大

它带来的直觉收益是：

- token 可以由更灵活的 expert 组合来服务
- 复杂知识更容易被拆散到不同专家上
- expert specialization 更容易形成

这里最值得看的图是 `Figure 2`。  
它把三步放在一张图里：

- 传统 top-2 routing
- 加上 finer-grained segmentation
- 再加上 shared expert isolation

这张图基本就是第二阶段 FFN 这条线的第一张总图。

#### 3.2 Shared Expert Isolation

只把 expert 切细还不够，因为 routed experts 之间还可能重复学公共知识。

所以作者做了第二个动作：

- 单独划出一部分 `shared experts`
- 它们不走 router，而是总会被激活
- 用来吸收跨上下文共通的知识

这样做的目标是：

- 让 routed experts 更专注于差异化知识
- 减少 routed experts 之间的参数冗余

一句人话总结：

> shared experts 像一层“公共知识底座”，  
> routed experts 才更容易真正做专门分工。

#### 3.3 Load Balance Consideration

这节说明作者并没有把问题停在算法层，而是立刻面对 MoE 的老难题：负载不均衡。

这里主要有两层 balance：

- `expert-level balance loss`
- `device-level balance loss`

对应两个现实问题：

- routing collapse：少数 experts 被疯狂选中，其他 experts 训练不起来
- 多机多卡部署时，某些设备成为瓶颈

这也是第二阶段一个非常典型的 DeepSeek 风格：

> 新架构一提出，立刻就把“训练时会不会炸、部署时会不会卡”一起纳入设计。

### 4. Validation Experiments：先在 2B 规模上证明这个方向成立

这一章的任务不是追求绝对最强模型，而是验证 `DeepSeekMoE` 这个架构本身是否值得继续扩大。

论文的验证逻辑是：

- 先做 `2B` 规模
- 和传统 `GShard` 风格的 MoE 比
- 也和相同总参数量的 dense 模型比

这里最值得记的几条结论：

- `DeepSeekMoE 2B` 明显强于 `GShard 2B`
- 它能接近 `GShard 2.9B`
- 它甚至接近相同总参数量 dense 模型的上限表现

这一步的意义非常大，因为它说明：

> DeepSeekMoE 不是“省算但掉能力”的妥协版，  
> 它是在省算的同时，尽量逼近 dense 模型上界。

### 5. Scaling up to DeepSeekMoE 16B：从验证架构走向真实规模

这一章是第二阶段里的一个过桥点。

到这里，作者不再只证明“这个 MoE 结构有道理”，而是开始展示它能不能被训练成真正有竞争力的模型。

论文给出的关键信息包括：

- `16B` total parameters
- 训练在 `2T` tokens 上进行
- 和 `DeepSeek 7B`、`LLaMA2 7B` 做比较

结果主线是：

- 只用大约 `40%` 的计算量
- 做到和 `DeepSeek 7B`、`LLaMA2 7B` 可比的性能

这一步很关键，因为它让第二阶段从“架构 paper”开始变成“路线 paper”。

### 6. Alignment for DeepSeekMoE 16B：MoE 不只是 base model

这节的重要性经常被低估。

它回答的是：

> 这种更复杂的 MoE 架构，能不能像 dense 模型一样顺利做对齐和 chat 化？

论文的答案是可以。

这意味着 `DeepSeekMoE` 不只是一个学术原型，而是有机会进入真实产品链路。

### 7. DeepSeekMoE 145B Ongoing：先放出扩展信号

这一节虽然还是 ongoing，但意义很清楚：

- 说明作者并不把 `16B` 当终点
- 他们已经在验证更大规模上的可扩展性

这也解释了为什么紧接着会出现 `DeepSeek-V2`：

> `DeepSeekMoE` 先把 FFN 侧的经济性和 specialization 跑通，  
> `V2` 再把它和注意力侧的优化合并成真正的一代模型。

### 第一篇最该带走的 6 个点

1. `DeepSeekMoE` 的核心不是“又一个 MoE”，而是“更强的 expert specialization”。
2. 论文点明了传统 MoE 的两个结构问题：knowledge hybridity 和 knowledge redundancy。
3. fine-grained expert segmentation 解决的是“expert 太粗、组合不灵活”。
4. shared expert isolation 解决的是“公共知识到处重复学”。
5. balance loss 说明作者从一开始就在把训练与部署约束写进架构设计。
6. `2B -> 16B -> 145B` 这条验证路径，说明它不是一次性 demo，而是一条要继续放大的路线。

## Part II. DeepSeek-V2

### 论文信息

- 标题：[DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model](https://arxiv.org/abs/2405.04434)
- arXiv：`2405.04434`
- 首版日期：`2024-05-07`
- 当前常被引用的版本：`v5`, `2024-06-19`
- 它在路线里的位置：`DeepSeekMoE + MLA`

### 原文目录地图

主干结构是：

1. `Introduction`
2. `Architecture`
   - `2.1 MLA`
   - `2.2 DeepSeekMoE`
3. `Pre-Training`
4. `Alignment`
5. `Conclusion, Limitation, and Future Work`

值得额外看的附录：

- `Appendix B`: `DeepSeek-V2-Lite`
- `Appendix D`: attention ablations，尤其是 `MHA / GQA / MQA / MLA` 的对比

### Abstract：这一代模型到底完成了什么

摘要已经把第二阶段的成果压得很清楚：

- `236B` total parameters
- `21B` activated parameters per token
- `128K` context
- `8.1T` pretraining tokens
- `SFT + RL`

最重要的是它明确把第二阶段的两个关键词写在一起：

- `DeepSeekMoE`
- `MLA`

这说明：

> `V2` 不是只有一个创新点，  
> 它是 FFN 稀疏化和注意力压缩第一次被完整合体。

### 1. Introduction：第二阶段的问题被重新定义了

Introduction 的第一步还是熟悉的背景：

- 模型越大，能力通常越强
- 但训练成本和推理吞吐会迅速恶化

然后作者马上把 `V2` 的问题定义清楚：

> 怎样在保持强能力的同时，把训练成本、KV cache 和推理吞吐一起优化？

这一节的关键贡献说明也很明确：

1. 注意力侧：`MLA`
2. FFN 侧：`DeepSeekMoE`
3. 系统侧：配套通信与训练优化

换句话说，第二阶段从一开始就不是“单点模型提分”，而是“整体经济性重构”。

### 2. Architecture：`MLA + DeepSeekMoE` 的真正合体

#### 2.1 MLA：为什么要从 MHA 走向 latent KV compression

这一节要解决的是推理时最现实的瓶颈之一：

> `MHA` 的 KV cache 太重了。

论文先把已有替代路线摆出来：

- `MQA`
- `GQA`

然后指出问题：

- 它们虽然省 cache
- 但会损失能力

所以 `MLA` 的目标不是只压缩，而是：

> 用 low-rank key-value joint compression，在尽量保住甚至超过 `MHA` 能力的前提下，把 KV cache 大幅压小。

这节里最值得看的表是 `Table 1`。  
它把不同注意力机制的 KV cache 规模放在一起比较，并给出一个很强的结论：

- `MLA` 的 KV cache 接近极小的分组水平
- 但能力目标不是退到 `MQA/GQA` 的妥协线，而是保持 strong 甚至 stronger

这就是第二阶段注意力线的本质：

> 不是简单“少存一点 KV”，而是“让长上下文和高吞吐真正可用”。

#### 2.2 DeepSeekMoE：为什么 V2 不是简单搬运前一篇论文

这一节在 `V2` 里不是重复 `DeepSeekMoE` 论文，而是把那套架构放进真实大模型训练系统里。

除了 basic architecture 外，它额外强调了几件事：

- `device-limited routing`
- `auxiliary loss for load balance`
- `token-dropping strategy`

也就是说，`V2` 里的 `DeepSeekMoE` 已经不是“概念版 MoE”，而是“能在真实集群上跑的大模型版 MoE”。

### 3. Pre-Training：第二阶段从架构进入大规模训练

这一章把第二阶段真正拉到了模型级别。

最重要的事实是：

- `V2` 预训练用了 `8.1T` tokens
- 模型支持 `128K` context
- 长上下文扩展不是原生从头训到 `128K`，而是先预训练，再用 `YaRN` 做扩展

这里有三个重点：

#### 3.1 Data Construction

作者沿用了 `DeepSeek 67B` 的数据处理阶段，但：

- 扩大了数据量
- 提升了数据质量
- 增加了中文数据
- 讨论了 pre-training data debiasing

这说明第二阶段虽然换了架构，但并没有把第一阶段对数据工程的重视扔掉。

#### 3.1.4 Long Context Extension

这节很值得单独记住。

它说明：

- `V2` 的默认 context 先从 `4K` 起步
- 再借助 `YaRN` 扩展到 `128K`
- 实际只额外训练了 `1000` steps、`32K` sequence length
- 但模型在 `128K` 的 NIAH 测试里仍然表现稳定

这部分对应第二阶段另一个非常现实的结论：

> 长上下文不只是模型结构问题，也是 curriculum 和 extension strategy 问题。

#### 3.2 Training and Inference Efficiency

这是 `V2` 最不能跳过的一节。

作者直接给出第二阶段最硬的工程结果：

- 相比 `DeepSeek 67B`，训练成本降低 `42.5%`
- 实际部署下 KV cache 降低 `93.3%`
- 单节点 `8 x H800` 上最大生成吞吐提升到 `5.76x`

这几组数字之所以重要，是因为它们把“经济、强、快”这三个词真正落到了可比较指标上。

### 4. Alignment：第二阶段开始显式引入 `SFT + RL`

和第一阶段相比，这里最值得注意的变化不是有没有 SFT，而是：

- `V2` 已经明确把 `RL` 写进正式 pipeline
- 这说明 DeepSeek 的后训练路线在第二阶段已经开始发生变化

#### 4.1 SFT

作者给出的核心设定包括：

- `1.5M` instruction tuning instances
- 其中 `1.2M` helpfulness，`0.3M` safety
- `2` epochs
- `5e-6` learning rate

这表明第二阶段并没有抛弃第一阶段的数据化、稳妥式 SFT，而是在更强 base 之上继续做高质量 instruction tuning。

#### 4.2 RL

这里有两个层次值得记：

1. 方法层：第二阶段已经开始把 RL 视为标准后训练组成部分。
2. 工程层：作者专门写了 RL 训练效率优化，包括：
   - train/inference different parallel strategies 的 hybrid engine
   - 用 `vLLM` 做大 batch inference backend
   - CPU/GPU offload scheduling

这一节很关键，因为它预告了后面 `R1` 那条线并不是凭空出现的。

#### 4.4 Discussion

`V2` 的 discussion 很有价值，至少有两条：

- `SFT` 数据量不能想当然地无限压缩，足够的数据量仍然重要
- `RL` 会带来 alignment tax，需要在开放式对话质量与标准 benchmark 之间做平衡

这说明第二阶段对齐线的主题已经不再是“要不要 RL”，而是“怎么让 RL 带来收益，同时控制代价”。

### 5. Conclusion：第二阶段正式收口

到 conclusion，这一代模型的定位已经很明确：

- 架构上：`DeepSeekMoE + MLA`
- 训练上：强 base + 经济训练
- 推理上：极大压缩 KV cache、提升吞吐
- 对齐上：`SFT + RL`

它的价值不只是单模型成绩，而是把第二阶段的整体方向钉死了：

> 继续做大模型，不必默认等于更贵、更慢、更难部署。

### 第二篇最该带走的 7 个点

1. `V2` 把第二阶段真正定义成“经济性重构”，而不是单点提分。
2. `MLA` 是第二阶段注意力线的核心，因为它直面 KV cache 瓶颈。
3. `DeepSeekMoE` 在 `V2` 里从架构想法进化成了真实训练系统的一部分。
4. `8.1T` tokens 和 `128K` context 说明第二阶段已经进入更大规模预训练和长上下文阶段。
5. `YaRN` 扩展表明长上下文不一定全靠从头原生训练，也可以通过扩展策略完成。
6. `SFT + RL` 说明后训练路线已经从第一阶段的 `SFT + DPO` 往后移动。
7. 训练成本、KV cache 和吞吐这三组工程指标，是理解 `V2` 最不能跳过的部分。

## 第二阶段合起来，到底学到了什么

如果把 `DeepSeekMoE` 和 `DeepSeek-V2` 合起来看，第二阶段最本质的收获是：

1. 第一阶段解决的是“训练规律”，第二阶段解决的是“经济扩展”。
2. FFN 这一侧的答案是：`DeepSeekMoE`，靠更细粒度 expert 和 shared experts 提高 specialization。
3. 注意力这一侧的答案是：`MLA`，靠 latent KV compression 压低 cache 成本。
4. 第二阶段的创新不是只停在论文公式里，而是直接落到了通信、负载均衡、token dropping、长上下文扩展和 RL 训练优化上。
5. 从这里开始，DeepSeek 路线已经不再只是“会不会训大模型”，而是“能不能把大模型训得强、训得便宜、跑得起来”。

## 如何回到路线笔记

回到 [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md) 时，可以只记这两条：

- `FFN -> DeepSeekMoE`
- `MHA -> MLA`

它们共同定义了 DeepSeek 的第二阶段：

> 不再只是把模型做大，  
> 而是把“继续做大”变成一件经济、可部署、可持续扩展的事。

## 相关笔记

- [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md)
- [DeepSeek LLM 论文阅读笔记](deepseek-llm-v1-paper-reading.md)
