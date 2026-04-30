# DeepSeek LLM 论文阅读笔记

- Created: 2026-04-30
- Updated: 2026-04-30
- Type: learning
- Status: draft
- Tags: deepseek, deepseek-llm, scaling-law, pretraining, alignment, paper-reading
- Model: GPT-5.4
- Harness: Codex
- Source: close reading of the DeepSeek LLM paper, rewritten section by section against the original paper structure

## 背景

这篇笔记只做一件事：

> 按 `DeepSeek LLM` 论文原文的目录和论证顺序，把作者到底说了什么、每一节承担什么功能、关键图表在支撑什么结论，重新梳理清楚。

它和路线笔记 [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md) 的分工是：

- 路线笔记：负责讲 `DeepSeek LLM / V1 -> V4` 的整体演化
- 这篇笔记：只负责第一阶段论文本身，不重复后续版本故事

## 这篇笔记怎么用

建议按下面的顺序读：

1. 先看每一节的“这一节在原文里做什么”
2. 再看“按段落顺序梳理”
3. 最后看“关键图表与表格”

这篇笔记刻意不把后续 `V2 / V3 / R1 / V4` 展开进来，只在每一节最后用一句话提醒它和路线笔记的关系。

## 论文信息

- 标题：[DeepSeek LLM: Scaling Open-Source Language Models with Longtermism](https://arxiv.org/abs/2401.02954)
- arXiv：`2401.02954`
- 首版日期：`2024-01-05`
- 对应路线位置：[DeepSeek V1-V4 研究路线笔记中的第一阶段](deepseek-v1-to-v4-research-route.md)

## 原文目录地图

论文正文结构是：

1. `Introduction`
2. `Pre-Training`
   - `2.1 Data`
   - `2.2 Architecture`
   - `2.3 Hyperparameters`
   - `2.4 Infrastructures`
3. `Scaling Laws`
   - `3.1 Scaling Laws for Hyperparameters`
   - `3.2 Estimating Optimal Model and Data Scaling`
   - `3.3 Scaling Laws with Different Data`
4. `Alignment`
5. `Evaluation`
   - `5.1 Public Benchmark Evaluation`
   - `5.1.1 Base Model`
   - `5.1.2 Chat Model`
   - `5.2 Open-Ended Evaluation`
   - `5.2.1 Chinese Open-Ended Evaluation`
   - `5.2.2 English Open-Ended Evaluation`
   - `5.3 Held-Out Evaluation`
   - `5.4 Safety Evaluation`
   - `5.5 Discussion`
6. `Conclusion, Limitation, and Future Work`

当前最值得重点看的附录部分：

- `A.2 Different Model Scale Representations`
- `A.3 Benchmark Metrics Curves`

## Abstract

### 这一节在原文里做什么

摘要不是在展开所有技术细节，而是在快速回答 4 个问题：

- 这篇论文想解决什么问题
- 做了哪些事
- 训练出了什么模型
- 结果大致如何

### 按原文信息顺序梳理

1. 前两句先定背景：开源 LLM 发展很快，但以往 scaling law 文献给出的结论并不一致。
2. 接着点出本文任务：重新研究 scaling laws，并给出适用于 `7B` 和 `67B` 开源配置的结论。
3. 然后交代工程落地：构建 `2T` bilingual corpus，从零训练 base model，再通过 `SFT + DPO` 做 chat model。
4. 最后交代结果：`67B` base 在很多 benchmark 上超过 `LLaMA-2 70B`，尤其是 code、math、reasoning；`67B Chat` 在中英文 open-ended evaluation 中强于 `GPT-3.5`。

### 这一节应该怎么理解

摘要真正传达的是：

- 这不是一篇“只发了一个新模型”的论文
- 也不是一篇“只讲 scaling 拟合”的论文
- 它是“用 scaling law 指导真实训练决策，并把结果训练出来”的论文

## 1. Introduction

### 这一节在原文里做什么

Introduction 的任务不是介绍模型参数，而是回答：

> 为什么在已经有 Kaplan 和 Chinchilla 的情况下，DeepSeek 还要重新做一遍 scaling law 研究？

### 按段落顺序梳理

#### 段落 1：先给大背景

作者先回到最基础的共识：

- decoder-only Transformer 已经是现代 LLM 的核心底座
- ChatGPT、Claude、Bard 让大家对大模型能力的预期迅速提高

这一段的功能是把论文放进整个 LLM 发展浪潮里。

#### 段落 2：再给开源背景

作者接着说开源生态：

- `LLaMA` 成了开源社区的默认比较基线
- 但开源社区的注意力更多放在“做一个固定规模的高质量模型”
- 对“如何系统性扩展模型”这件事，关注还不够

这一段把问题从“大模型很火”收缩到“开源模型如何系统扩展”。

#### 段落 3：引出核心动机

这一段是 Introduction 最关键的动机段。

作者指出：

- 早期 scaling law 工作给出的最优 model/data allocation 结论并不一致
- 很多工作没有把 hyperparameter 本身的 scaling 研究完整
- 因此，不同 compute budget 下模型是否真的训练到了近最优，未必能保证

这也是整篇论文真正的起点：

> 他们不满足于照搬既有 scaling 结论，而是要把“训练配方本身”也纳入 scaling 研究。

#### 段落 4：第一个贡献

第一点贡献是：

- 他们先研究 `batch size` 和 `learning rate` 的 scaling laws
- 并发现这两个最关键超参数会随模型规模和 compute budget 呈现稳定趋势

这是你提到的第一点，确实属于 Introduction，而不是摘要。

#### 段落 5：第二个贡献

第二点贡献是：

- 他们系统研究了 model scale 和 data scale 的 scaling law
- 给出 compute budget 固定时更优的 model/data allocation 策略
- 还能据此预测更大模型的预期性能

这里的重点不是“他们训练了大模型”，而是“他们试图让大模型训练前就可规划”。

#### 段落 6：第三个贡献

第三点贡献是：

- 不同数据集会导出显著不同的 scaling law
- 数据集选择会明显影响 scaling behavior
- 因此，不能把某个数据集上拟合出来的 scaling law 无条件推广到别处

这一点很重要，因为它把“数据质量”从配角提升成了会改写 scaling 结论的因素。

#### 段落 7：第四个贡献，也是工程落地部分

这一段开始讲他们具体做了什么：

- 在 scaling law 指导下，从零训练 `7B` 和 `67B` 基础模型
- 构建 `2T` bilingual corpus
- 架构总体跟随 `LLaMA`
- 学习率调度不用 cosine，而改用 multi-step scheduler
- 收集超过 `1M` 的 SFT 数据
- 做 DPO
- 还分享不同 SFT 策略和数据消融的经验

也就是说，Introduction 不只是“列理论贡献”，还提前交代了这是一篇研究和工程一起落地的论文。

#### 段落 8：结果概览

这里作者先给读者看结果大图：

- `DeepSeek 67B` 在很多 benchmark 上超过 `LLaMA-2 70B`
- 优势尤其集中在 code、math、reasoning
- `DeepSeek 67B Chat` 在中英文 open-ended evaluation 中优于 `GPT-3.5`
- 安全评测也显示它能在实践中给出较安全的回答

这一段的作用是提前告诉你：前面那些 scaling、数据和训练配方研究，不是纸上谈兵。

#### 段落 9：论文结构说明

最后一段是标准的 roadmap：

- Section 2 讲 pre-training 的基本设置
- Section 3 讲 scaling laws
- Section 4 讲 alignment
- Section 5 讲 evaluation
- Section 6 讲 limitation 和 future work

### 小结

Introduction 真正做了四件事：

- 给背景
- 给动机
- 给贡献列表
- 给结果预告和全文导航

如果这一节读顺了，后面就不会把整篇论文误读成“纯模型发布报告”。

## 2. Pre-Training

### 这一章在原文里做什么

Section 2 不是在追求炫技，而是在把一个问题讲清楚：

> 在真正进入 scaling law 拟合和大规模训练前，DeepSeek 的基础训练盘子到底是怎么搭的？

它分成四块：

- 数据
- 架构
- 超参数
- 基础设施

### 2.1 Data

#### 这一小节在原文里做什么

回答的是：

> 这套 `2T` bilingual 预训练数据，具体是怎么整理出来的？

#### 按段落顺序梳理

##### 段落 1：先讲数据处理目标和三阶段流程

作者先说数据处理目标不是简单“变大”，而是：

- 提高 richness
- 提高 diversity
- 提高 information density

然后把流程拆成三步：

- `deduplication`
- `filtering`
- `remixing`

这里已经能看到他们的数据工程思路：

- deduplication/remixing 负责“覆盖更广、更不重复”
- filtering 负责“信息密度更高”

##### 段落 2：重点讲 aggressive deduplication

作者强调他们采用了更激进的去重策略，并扩大了去重范围。

最重要的论点是：

- 在整个 Common Crawl 语料层面做去重
- 比只在单个 dump 内去重，能去掉更多重复内容

##### `Table 1` 在说明什么

`Table 1` 给的是不同 Common Crawl dump 范围下的去重率：

- `1` dump：`22.2%`
- `91` dumps：`89.8%`

这个表的价值不是记住每个数字，而是确认作者的论点：

> 去重范围不是细节，它会直接决定最终训练语料的重复污染程度。

##### 段落 3：过滤阶段

作者说 filtering 的重点是建立 robust 的文档质量评估标准，包括：

- linguistic evaluation
- semantic evaluation
- individual perspective
- global perspective

换句话说，他们不是只用浅层规则清洗，而是把数据质量看成一个多维判断问题。

##### 段落 4：重混阶段

remixing 的目的是解决数据分布不平衡：

- 提高代表性不足领域的占比
- 让整体数据分布更平衡
- 尽量覆盖更丰富的知识和视角

这里的重点是：他们已经在用“数据再配比”去影响模型后续学到什么。

##### 段落 5：tokenizer

最后一段交代 tokenizer：

- 使用 `BBPE`
- 采用 pre-tokenization，防止不同字符类别随意合并
- 数字拆成单独 digit
- 常规词表大小设为 `100000`
- 最终加上 `15` 个 special tokens 后，总词表 `100015`
- 为了训练效率和未来扩展，训练时词表大小配置为 `102400`

#### 小结

这节的重点不是“他们也用了 BBPE”，而是：

- 数据工程是严肃主线
- 去重范围、质量过滤、分布重混都会影响后面的 scaling 行为

### 2.2 Architecture

#### 这一小节在原文里做什么

回答的是：

> 在第一阶段还没有 `MoE / MLA` 的情况下，DeepSeek 选了什么样的 dense Transformer 底座？

#### 按段落顺序梳理

##### `Table 2`：先把关键规格放出来

`Table 2` 给出两组模型的关键配置：

- `7B`：`30` 层，`d_model=4096`，`32` heads，`32` kv heads，`4096` context，`2304` sequence batch，`4.2e-4` learning rate，`2.0T` tokens
- `67B`：`95` 层，`d_model=8192`，`64` heads，`8` kv heads，`4096` context，`4608` sequence batch，`3.2e-4` learning rate，`2.0T` tokens

这个表的重要性在于：

- 它不是孤立配置表
- 它是后面 scaling law 结论落到真实模型上的结果

##### 段落 1：micro design 基本沿用 LLaMA

作者先说明微观架构上总体跟随 `LLaMA`，包括：

- `Pre-Norm`
- `RMSNorm`
- `SwiGLU`
- `RoPE`

这一步是在告诉读者：

- 第一阶段不是靠大改架构取胜
- 而是先在成熟 dense 架构上把训练规律做扎实

##### 段落 2：67B 使用 GQA

为了降低推理成本，`67B` 使用了 `GQA`，而不是传统 `MHA`。

这个动作很小，但很关键，因为它说明：

- 即使在第一阶段
- DeepSeek 也已经开始关心推理侧成本，而不是只看预训练结果

##### 段落 3：macro design 与 LLaMA 略有不同

作者说明在宏观层面做了调整：

- `7B` 采用 `30` 层
- `67B` 采用 `95` 层

调整原因除了参数规模，还包括：

- 方便 pipeline partition
- 优化训练和推理

##### 段落 4：67B 选择“更深”而不是“更宽”

作者特别强调：

- 他们不是主要通过加宽 FFN 中间层来扩参数
- 而是把模型做得更深

这是一个很值得记的选择，因为它预示了后续 DeepSeek 一贯的工程取向：

- 不只是“参数更多”
- 还要考虑训练、推理、切分和系统成本

#### 小结

这一节最重要的不是记住每个超参数，而是理解：

- 第一阶段的架构创新并不激进
- 重点是选一个稳定且可继续扩展的 dense 底座

### 2.3 Hyperparameters

#### 这一小节在原文里做什么

回答的是：

> 这套基础模型在真正训练时，默认超参数配方是什么？为什么这样选？

#### 按段落顺序梳理

##### 段落 1：基础优化器配置

作者先交代：

- 初始化标准差：`0.006`
- 优化器：`AdamW`
- `beta1 = 0.9`
- `beta2 = 0.95`
- `weight_decay = 0.1`

这部分更多是给 reproducibility 和后文 Section 3 做铺垫。

##### 段落 2：学习率调度器的核心设定

这里是本节最重要的地方。

作者明确说：

- 不用典型 cosine scheduler
- 改用 `multi-step learning rate scheduler`

具体是：

- `2000` warmup steps 后到达峰值
- 处理完 `80%` tokens 后衰减到峰值的 `31.6%`
- 处理完 `90%` tokens 后衰减到峰值的 `10%`
- gradient clipping 设为 `1.0`

##### 段落 3：`Figure 1(a)` 的作用

`Figure 1(a)` 比较的是：

- multi-step scheduler
- cosine scheduler

作者想说明的不是“multi-step 一定更强”，而是：

- 两者最终表现基本一致
- 但 multi-step 更利于 continual training 和阶段复用

##### 段落 4：`Figure 1(b)` 的作用

`Figure 1(b)` 比较的是 multi-step 各阶段比例。

作者的结论是：

- 某些比例会略好
- 但他们最后选择的是在性能和复用性之间更平衡的 `80% / 10% / 10%`

##### 段落 5：把它和模型规模联系起来

最后一句收口：

- batch size 和 learning rate 会随模型规模变化
- 具体 7B/67B 的参数放在 `Table 2`

这句其实是在把 Section 2 和 Section 3 接起来：前面是实际配置，后面会解释这些配置的 scaling 逻辑。

#### 小结

这一节最值得带走的是：

- DeepSeek 在第一阶段已经很强调“训练策略可复用”
- 不是只看一次性训练是否略优

### 2.4 Infrastructures

#### 这一小节在原文里做什么

回答的是：

> 这些实验和训练不是在真空里完成的，它们具体依赖什么训练系统和基础设施？

#### 按段落顺序梳理

##### 段落 1：训练框架和并行策略

作者先介绍自研框架 `HAI-LLM`，并说明集成了：

- data parallelism
- tensor parallelism
- sequence parallelism
- `1F1B` pipeline parallelism

##### 段落 2：性能与稳定性优化

这里列的是一整串系统优化：

- `FlashAttention`
- `ZeRO-1`
- 计算与通信重叠
- fused operators
- `bf16` 训练但 `fp32` 累积梯度
- inplace cross-entropy 节省显存

这一段的功能是告诉你：

- 论文虽然主线是 scaling law
- 但作者并没有把系统工程当成背景噪音

##### 段落 3：checkpoint 与恢复机制

作者专门提到：

- 每 `5` 分钟异步保存模型和优化器状态
- 最坏情况下只会损失不超过 `5` 分钟训练
- 支持在不同 `3D parallel configuration` 下恢复训练

这已经明显超出“普通论文配置说明”的范围，更像是生产级训练系统经验。

##### 段落 4：评测基础设施

最后补充：

- 生成类任务用 `vLLM`
- 非生成类任务用 continuous batching

目标是减少手工 batch 调参和 padding 浪费。

#### 小结

这一节虽然短，但很关键：

- DeepSeek 从第一阶段开始就把“训练系统”当成主线的一部分
- 后面 `V3 / V4` 的系统能力，不是突然出现的

## 3. Scaling Laws

### 这一章在原文里做什么

Section 3 是全篇核心。它真正回答的是：

> 如果想长期、持续地把模型做大，compute budget、model scale、data scale、batch size、learning rate 到底该怎么一起配？

### 先看 3.0 这段总引子

#### 按段落顺序梳理

##### 段落 1：回顾 scaling law 的经典表述

作者先回顾：

- scaling law 早于 LLM 爆发
- 经典研究认为性能会随着 `compute C`、`model scale N`、`data scale D` 的增加而可预测提升
- 当 `N` 用参数量表示、`D` 用 token 数表示时，通常用 `C = 6ND` 近似

##### 段落 2：为什么 LLM 时代让 scaling law 更重要

因为更大的模型不断带来显著收益，所以：

- 大家更愿意继续加 compute
- model/data 的分配问题也就更重要

##### 段落 3：为什么要“重做” scaling law

作者指出两类问题：

- 以往最优 model/data allocation 结论不一致
- 很多工作没有完整描述超参数设定，因此不确定不同 compute budget 下是否真训练到了最优

这就是本章存在的根本理由。

##### 段落 4：先研究 hyperparameter scaling

作者说大多数超参数可以保持不变，但两个最关键的量不行：

- `batch size`
- `learning rate`

因此要先研究这两个量随 compute budget 的变化规律，保证不同预算下都能训练到 near-optimal。

##### 段落 5：再研究 model/data scaling

这里引出第二个核心动作：

- 用 `IsoFLOP` profile 拟合 scaling curve
- 不再只用参数量 `N` 表示模型规模
- 改用 `M = non-embedding FLOPs/token`

##### 段落 6：最后引入数据质量问题

作者指出：

- 不同数据集会导出不同 scaling law
- 数据质量可能正是过去结论差异很大的原因之一

### 小结

3.0 这段引子实际上已经把整章结论预告完了：

- 先校准超参数 scaling
- 再校准 model/data scaling
- 最后把数据质量纳入解释框架

### 3.1 Scaling Laws for Hyperparameters

#### 这一小节在原文里做什么

回答的是：

> `batch size` 和 `learning rate` 能不能像 model/data 一样，呈现稳定的 scaling 规律？

#### 按段落顺序梳理

##### 段落 1：先做小规模网格搜索

作者先在 `1e17 FLOPs` 的 compute budget 上，对 `batch size` 和 `learning rate` 做 grid search。

目的很明确：

- 不先假设经验公式一定适用
- 而是先看自己的训练分布上最优区域长什么样

##### 段落 2：`Figure 2(a)` 的核心信息

`Figure 2(a)` 展示的是：

- generalization error 在一片相当宽的超参数区域里都比较稳定

这一点很重要，因为它说明：

- near-optimal 区域不是一个“神秘点”
- 而是一条可操作的带状区域

##### 段落 3：跨 compute budget 训练更多点

作者接着在 `1e17` 到 `2e19` FLOPs 的多个预算上训练模型，并利用 multi-step scheduler 的可复用性降低实验成本。

##### 段落 4：给 near-optimal 下定义

他们把 near-optimal 定义为：

- generalization error 距离最小值不超过 `0.25%`

这是个很有用的工程定义，因为它承认真实训练并不需要击中一个单点最优值。

##### 段落 5：拟合出两个幂律公式

作者给出：

- `η_opt = 0.3118 * C^-0.1250`
- `B_opt = 0.2920 * C^0.3271`

其中：

- compute 越大，最优 learning rate 越小
- compute 越大，最优 batch size 越大

##### 段落 6：`Figure 3` 的作用

`Figure 3` 把这两个 scaling curve 画出来，并用：

- 灰点表示 near-optimal 区域
- 虚线表示幂律拟合
- 蓝色星标标出 7B / 67B 的取值

这张图的作用是说明：

- 小规模实验拟合出来的规律
- 可以实际指导大模型训练配置

##### 段落 7：额外验证和边界条件

作者还在更高 compute budget 上做了验证，并提醒：

- compute budget 不是唯一因素
- 即使 compute 相同，不同 model/data allocation 也会让最优超参数空间略有变化

这使他们的表述更谨慎，没有把拟合公式说成放之四海而皆准。

#### 关键图表与公式

`Figure 2`

- 看点：near-optimal 区域是宽带，不是尖点
- 含义：超参数 scaling 可以做成实用的搜索框架，而不是玄学

`Figure 3`

- 看点：batch size 和 learning rate 的幂律拟合，以及 7B/67B 的落点
- 含义：小实验可以外推出真实大模型训练配方

#### 小结

这节真正建立的是：

- 超参数也可以有 scaling law
- 而且这些规律足够稳定，可以直接服务训练决策

### 3.2 Estimating Optimal Model and Data Scaling

#### 这一小节在原文里做什么

回答的是：

> 给定 compute budget，模型和数据应该怎么一起放大才更优？

#### 按段落顺序梳理

##### 段落 1：先提出问题形式

作者想求的是：

- 在 `C` 固定时
- 选择什么样的 `M` 和 `D`
- 才能让 generalization error 最小

这里和传统写法不同的关键，在于他们不用 `N`，而改用 `M`。

##### 段落 2：为什么不用参数量 `N`

作者讨论了三种模型规模表示法：

- `6N1 = 72 n_layer d_model^2`
- `6N2 = 72 n_layer d_model^2 + 6 n_vocab d_model`
- `M = 72 n_layer d_model^2 + 12 n_layer d_model l_seq`

其中 `M` 表示 `non-embedding FLOPs/token`。

他们的观点是：

- 单纯看参数量会低估或高估不同结构下的真实计算成本
- 尤其在不同规模区间，这种偏差会更明显

##### `Table 3` 在说明什么

`Table 3` 的作用不是展示一个漂亮公式，而是支撑这句话：

> “模型规模”如果定义得不对，后面的 scaling law 拟合本身就会偏。

##### 段落 3：正式给出优化目标

作者把问题写成：

- 在 `C = MD` 约束下
- 最小化 generalization error

这个写法的意义是把原本模糊的“怎么分配模型和数据”变成一个明确优化问题。

##### 段落 4：实验方法

他们采用 `Chinchilla` 的 `IsoFLOP` profile 思路：

- 选 `8` 个 compute budget，从 `1e17` 到 `3e20`
- 每个 budget 下测试约 `10` 组 model/data allocation

这样可以在控制成本的前提下拟合出最优曲线。

##### 段落 5：拟合最优 model/data scaling

他们得到：

- `M_opt = 0.1715 * C^0.5243`
- `D_opt = 5.8316 * C^0.4757`

也就是：

- compute 增加时，模型和数据都应扩展
- 但在他们当前数据分布下，模型扩展指数略大于数据扩展指数

##### 段落 6：`Figure 4` 的作用

`Figure 4` 展示了：

- IsoFLOP 曲线
- 最优 model scaling
- 最优 data scaling

它的价值是让“allocation strategy”不再停留在口头描述，而变成一条可用曲线。

##### 段落 7：`Figure 5` 的作用

`Figure 5` 展示的是：

- 用前面小规模实验拟合出来的 scaling curve
- 去预测更大模型的性能

作者要证明的不是“预测绝对精确”，而是：

- 这种方法足以给大规模训练前的规划提供可信指导

#### 关键图表与公式

`Table 3`

- 看点：`6N1`、`6N2` 和 `M` 这三种模型规模表示法的区别
- 含义：先把“模型规模怎么度量”定义对，再谈 scaling law

`Figure 4`

- 看点：IsoFLOP curve 与最优 model/data 曲线
- 含义：allocation 可以被拟合成一条可用的规划曲线

`Figure 5`

- 看点：用拟合曲线预测大模型性能
- 含义：scaling law 可以从分析工具变成训练前决策工具

#### 小结

这一节的核心不是“大模型该多大”，而是：

- 如何重新定义模型规模
- 如何在固定算力下决定模型和数据的扩展比例

### 3.3 Scaling Laws with Different Data

#### 这一小节在原文里做什么

回答的是：

> 如果数据集质量和分布变了，最优的 model/data scaling 还会不会跟着变？

#### 按段落顺序梳理

##### 段落 1：为什么能研究这个问题

作者说在开发过程中，他们的数据集被反复迭代过：

- 数据源比例调整过
- 整体质量也提升过

所以现在有条件比较不同数据版本下的 scaling law。

##### 段落 2：比较三种数据集

他们比较了三套数据：

- early in-house data
- current in-house data
- `OpenWebText2`

并给出一个主观质量判断：

- current in-house 好于 early in-house
- `OpenWebText2` 由于规模更小、处理更细，质量还高于 current in-house

##### `Table 4` 在说明什么

`Table 4` 列的是不同数据分布下的 scaling 指数：

- `OpenAI (OpenWebText2)`：`a=0.73, b=0.27`
- `Chinchilla (MassiveText)`：`a=0.49, b=0.51`
- `Ours (Early Data)`：`a=0.450, b=0.550`
- `Ours (Current Data)`：`a=0.524, b=0.476`
- `Ours (OpenWebText2)`：`a=0.578, b=0.422`

这张表是全篇最值得反复看的表之一，因为它直接支撑作者的核心发现。

##### 段落 3：作者给出的主要发现

他们发现：

- 数据质量越高，模型 scaling 指数 `a` 越大
- 数据 scaling 指数 `b` 越小

也就是说：

- 数据越好，新增 compute 越值得分给模型
- 而不是一味继续堆更多数据

##### 段落 4：作者的直觉解释

作者给出的解释是：

- 高质量数据通常逻辑更清晰
- 在充分训练后，它的预测难度更低
- 因此继续增大模型，更容易吃到新增算力收益

这不是严格证明，但作为直觉解释很有帮助。

#### 小结

这一节把一个很重要的观念钉死了：

> scaling law 不是和数据无关的自然常数，数据质量本身会改写最优扩展策略。

## 4. Alignment

### 这一章在原文里做什么

Section 4 讨论的是：

> 预训练 base model 之后，作者如何把它变成一个更可用、更安全的 chat model？

### 按段落顺序梳理

#### 段落 1：先交代数据规模和分布

作者先给出对齐数据总量：

- 约 `1.5M` instruction data

其中：

- `1.2M` helpfulness data
- `300K` safety data

helpfulness 数据分布：

- `31.2%` general language
- `46.6%` math
- `22.2%` coding

这一段的作用是让读者知道：

- 他们的后训练数据并不是纯聊天数据
- math 和 coding 占比非常高

#### 段落 2：对齐流程只有两步

作者明确说：

- 整个 alignment pipeline 只有两个 stage
- 即 `SFT` 和 `DPO`

这也说明第一阶段的后训练仍然是相对传统的范式。

#### 段落 3：SFT 的训练设定

作者具体给出：

- `7B` 做 `4` epochs
- `67B` 只做 `2` epochs

原因是：

- 他们观察到 `67B` 更容易过拟合

并且：

- `7B` 的 `learning rate = 1e-5`
- `67B` 的 `learning rate = 5e-6`

#### 段落 4：为什么要监控 repetition

除了 benchmark accuracy，作者还监控：

- repetition ratio

他们收集了 `3868` 个中英文 prompt，测量模型是否会“无法停下并重复一段文本”。

这是很重要的实践细节，因为它说明他们没有只盯公开 benchmark。

#### 段落 5：math SFT 数据的副作用

作者观察到：

- math SFT 数据占比越高
- repetition ratio 越容易上升

他们给出的解释是：

- 数学推理数据里经常有相似 reasoning pattern
- 较弱模型不能真正掌握它们，只会学到重复模式

#### 段落 6：为了解决 repetition，尝试两种方法

作者说他们尝试了：

- two-stage fine-tuning
- `DPO`

结论是：

- 两者都能在基本不伤 benchmark 的情况下显著降低 repetition

#### 段落 7：DPO 数据构造

作者说明 DPO 数据从两方面构造：

- helpfulness
- harmlessness

helpfulness preference data 包含：

- creative writing
- question answering
- instruction following

并以 DeepSeek 自己的 chat model 输出作为 response candidates。

#### 段落 8：DPO 训练设定和效果

他们的 DPO 训练设定是：

- 训练 `1` epoch
- `learning rate = 5e-6`
- `batch size = 512`
- 使用 warmup 和 cosine scheduler

作者的总结很明确：

- DPO 可以增强 open-ended generation
- 但对标准 benchmark 的影响不大

### 小结

这一章最值得记的是：

- 第一阶段的后训练主线仍然是 `SFT + DPO`
- 但作者已经在用非常工程化的指标去管理副作用，比如 repetition

## 5. Evaluation

### 这一章在原文里做什么

Section 5 不是简单贴分数，而是在分层回答：

- base model 基础能力如何
- chat model 对齐后能力如何变化
- open-ended 对话质量如何
- held-out 泛化如何
- 安全性如何
- 实践上还观察到了什么经验

### 5.1 Public Benchmark Evaluation

#### 这一小节在原文里做什么

先定义评测覆盖面和评测方式，避免后面表格脱离上下文。

#### 按段落顺序梳理

##### 段落 1 到 9：列 benchmark 家族

作者按类别列了很多 benchmark：

- multi-subject multiple-choice：`MMLU`、`C-Eval`、`CMMLU`
- language understanding / reasoning：`HellaSwag`、`PIQA`、`ARC`、`OpenBookQA`、`BBH`
- closed-book QA：`TriviaQA`、`NaturalQuestions`
- reading comprehension：`RACE`、`DROP`、`C3`
- reference disambiguation：`WinoGrande`、`CLUEWSC`
- language modeling：`Pile`
- Chinese understanding / culture：`CHID`、`CCPM`
- math：`GSM8K`、`MATH`、`CMath`
- code：`HumanEval`、`MBPP`
- standardized exams：`AGIEval`

这一串列表的作用是说明：

- 评测面覆盖相当广
- 不只看英语，也不只看知识问答

##### 段落 10 到 12：讲三类评测方式

作者把 evaluation 分成三种：

- perplexity-based
- generation-based
- language-modeling-based

并明确说明每类 benchmark 用什么方式评测，以及是否用 greedy decoding、length normalization、unconditional normalization。

这一步是为了让后面分数可解释，而不是一张大表糊过去。

### 5.1.1 Base Model

#### 这一小节在原文里做什么

回答的是：

> 如果先不看对齐，只看预训练 base model，它在公开 benchmark 上处于什么水平？

#### 按段落顺序梳理

##### `Table 5`：主结果表

`Table 5` 是 base model 的总表。

作者从这张表里提炼出几个主要发现：

- 虽然 `DeepSeek` 是 `2T` bilingual corpus 训练的，但英语 benchmark 表现仍和 `LLaMA2` 可比
- `DeepSeek 67B` 在 `MATH`、`GSM8K`、`HumanEval`、`MBPP`、`BBH` 和中文 benchmark 上明显优于 `LLaMA2 70B`

##### 段落 1：同样 2T，双语没有把英语拖垮

这是作者很想强调的点：

- bilingual 预训练并没有显著牺牲英语理解能力

##### 段落 2：大模型在某些任务上的收益特别大

作者特别提到：

- `GSM8K`
- `BBH`

这些任务随模型规模增大提升明显。

他们给出的解释是：

- 这是大模型 few-shot learning 能力增强的体现

##### 段落 3：67B 相对 LLaMA2 的优势比 7B 更明显

作者认为这说明：

- language conflict 对小模型影响更大
- 模型大了之后，中英双语训练带来的冲突更容易被消化

##### 段落 4：能力跨语言迁移与语言专属知识

作者举了两个对照：

- 数学能力这类基础能力可以跨语言迁移，所以 `LLaMA2` 在某些中文数学任务上也不差
- 像 `CHID` 这种依赖中文成语知识的任务，就必须真正吃过大量中文语料

#### 小结

base model 部分的关键结论是：

- 第一阶段的双语预训练底座已经很强
- 而且不是“偏科型强”，而是在 code、math、reasoning 和中文上都显出特点

### 5.1.2 Chat Model

#### 这一小节在原文里做什么

回答的是：

> 经过 SFT / DPO 之后，chat model 相比 base model 到底得到了什么，又牺牲了什么？

#### 按段落顺序梳理

##### `Table 6`：base vs chat 对照表

这张表的作用非常直接：

- 把 `7B Base / 7B Chat / 67B Base / 67B Chat` 放在一起对比

##### 段落 1：总体变好，但不是所有任务都变好

作者先说总体上大多数任务都有提升，但也明确承认：

- 有些任务会下降

##### 段落 2：知识相关任务的波动怎么理解

作者点名：

- `TriviaQA`
- `MMLU`
- `C-Eval`

并说这些分数的轻微涨跌，不应被简单解释成“学到了知识”或“丢了知识”。

他们更看重的是：

- chat model 在 zero-shot 场景下
- 能接近 base model few-shot 的效果

也就是：

> SFT 的价值更偏“把能力变成可直接对话调用的形式”，而不只是纯知识增减。

##### 段落 3：reasoning 提升怎么理解

作者说由于很多 SFT 数据本身是 `CoT` 格式：

- chat model 在 `BBH`、`NaturalQuestions` 等 reasoning 任务上有轻微提升

但他们也非常谨慎：

- 他们不认为 SFT 真正教会了推理能力本身
- 更可能是教会了“如何用正确格式写出推理路径”

##### 段落 4：哪些任务会掉分

作者指出有些 consistently drop 的任务，通常是：

- cloze task
- sentence completion task

例如：

- `HellaSwag`

他们的解释是：

- 纯语言模型天然更适合这类任务
- 对齐后的 chat model 会为了交互而牺牲一部分这类能力

##### 段落 5：math 和 code 为什么提升大

作者特别指出：

- `HumanEval`
- `GSM8K`

fine-tuning 后常常能提升 `20+` 分。

他们给出的解释是：

- base model 对这些任务最初还偏 underfit
- 大量 math/coding SFT 数据补进了额外知识

但作者也加了一个重要保留：

- 这种提升可能偏向 code completion 和 algebraic questions
- 不代表已经形成更广义的数学和编程理解

##### 段落 6：two-stage SFT 在 7B 和 67B 上效果不同

作者最后补充一个重要实践观察：

- `7B` 第一阶段 SFT 后 repetition ratio 是 `2.0%`
- 第二阶段剔除 math/code 数据后降到 `1.4%`
- 且 benchmark 基本保持

但对 `67B`：

- 第一阶段后 repetition 已经低于 `1%`
- 再做第二阶段反而伤 benchmark

所以：

- `7B` 用了 two-stage SFT
- `67B` 只做一阶段 SFT

#### 小结

这节最有价值的地方在于作者没有把 fine-tuning 神化。他们明确区分了：

- 什么是对齐带来的真实收益
- 什么是任务格式适配
- 什么是新的 trade-off

### 5.2 Open-Ended Evaluation

#### 这一小节在原文里做什么

作者明确说：

- 标准 benchmark 很重要
- 但真正影响用户体验的是 open-ended generation

所以他们单独做了中英文 open-ended evaluation。

### 5.2.1 Chinese Open-Ended Evaluation

#### 按段落顺序梳理

##### 段落 1：数据集和评测方式

作者使用 `AlignBench`：

- `8` 个一级类别
- `36` 个二级类别
- `683` 个问题

并使用官方仓库和 `GPT-4` 评分模板进行评测。

##### 段落 2：温度设置细节

作者还专门说明：

- role-playing、writing、open-ended：temperature `0.7`
- 其他任务：temperature `0.1`

这类细节很值得记，因为 open-ended evaluation 很依赖生成设置。

##### `Table 7`：AlignBench leaderboard

作者从 `Table 7` 提炼出的结论是：

- `DeepSeek-67B-Chat` 明显强于 `ChatGPT` 和其他 baseline
- 总体只排在两个 `GPT-4` 版本后面
- `DPO` 在几乎所有维度都带来提升

##### 段落 3：中文基础语言能力

作者指出：

- 在 basic Chinese language tasks 上，DeepSeek 已在第一梯队
- `DPO` 版本甚至在这一维度高于最新 `GPT-4`

##### 段落 4：中文高级推理能力

作者也强调：

- 在更复杂的中文逻辑推理和数学任务上
- DeepSeek 的分数明显高于其他中文 LLM

#### 小结

中文 open-ended evaluation 的重点不是“总分第几”，而是：

- DeepSeek 第一阶段已经把中英文 chat 体验做到了非常强的水平
- 而且 `DPO` 的收益在真实对话类任务上很明显

### 5.2.2 English Open-Ended Evaluation

#### 按段落顺序梳理

##### 段落 1：评测集

作者使用 `MT-Bench`，包含 `8` 类多轮问题。

##### `Table 8`：MT-Bench 结果

作者给出的核心结论是：

- `DeepSeek LLM 67B Chat` 超过多种开源模型，如 `LLaMA-2-Chat 70B`、`Xwin 70B`、`TÜLU 2 + DPO 70B`
- 平均分 `8.35`，与 `GPT-3.5-turbo` 接近
- 做完 `DPO` 后提升到 `8.76`
- 仅次于 `GPT-4`

##### 段落 2：作者如何解释

作者把这一结果解读为：

- DeepSeek 的多轮 open-ended 生成能力已经很强
- `DPO` 在英语 open-ended 对话上也有明显正收益

#### 小结

英文 open-ended evaluation 的重点是：

- 标准 benchmark 之外，真实对话质量也做上去了
- 而 `DPO` 的最主要收益正体现在这里

### 5.3 Held-Out Evaluation

#### 这一小节在原文里做什么

回答的是：

> 如果不用那些大家都熟悉的 benchmark，而改用更晚、更新、更不容易污染的数据，模型还能不能站住？

#### 按段落顺序梳理

##### 段落 1：为什么要做 held-out evaluation

作者先指出两个经典问题：

- data contamination
- benchmark overfitting

所以需要新的 held-out testset。

##### 段落 2：LeetCode

他们用 `2023-07` 到 `2023-11` 的 LeetCode 周赛和双周赛题目：

- 共 `126` 题
- 每题 `20+` test cases
- 评价方式类似 `HumanEval`

##### 段落 3：Hungarian National High-School Exam

作者用匈牙利高中考试评估数学能力：

- 共 `33` 题
- 采用人工评分

##### 段落 4：Instruction Following Evaluation

他们还用 Google 发布的 instruction-following 数据集：

- `25` 类可验证指令
- 约 `500` prompts

##### `Table 9`：held-out 结果表

`DeepSeek LLM 67B Chat` 的结果：

- LeetCode：`17.5`
- Hungarian Exam：`58`
- IFEval：`55.5`

作者重点比较了：

- `Qwen 72B Chat`
- `ChatGLM3`
- `Baichuan2`
- `Yi-34B Chat`

##### 段落 5：作者的主要观察

他们指出：

- 大模型和小模型在 held-out 数据上的差距，比在传统 benchmark 上更明显
- 某些小模型在传统 benchmark 上看起来接近 DeepSeek，但换到更新测试集就明显掉队

##### 段落 6：7B vs 67B

作者还补了一个很有意思的观察：

- 7B 和 67B 训练流程相同
- 但主观上能明显感觉到 67B 在很多任务上的“智能感”更强

#### 小结

这一节最重要的意义是：

- 作者没有只拿公开 benchmark 自证
- 而是主动尝试检验“新题上的真实泛化”

### 5.4 Safety Evaluation

#### 这一小节在原文里做什么

回答的是：

> 模型除了有用之外，是否也足够安全？

#### 按段落顺序梳理

##### 段落 1：先讲原则

作者先表明立场：

- 真正有帮助的 AI，前提之一是价值观和人类一致，并对人类友好
- 安全性贯穿 pre-training、SFT、DPO 全流程

##### 段落 2：建立人工安全评测体系

他们组建了：

- `20` 人专家团队

并建立符合人类价值观的安全分类体系。

##### `Table 10`：安全分类法

`Table 10` 把安全问题分成几大类：

- 歧视偏见问题
- 侵犯他人合法权益
- 商业秘密与知识产权
- 违法违规行为
- 其他安全问题

并列出每类：

- 测试样本总数
- `DeepSeek-67B-Chat` 给出安全回答的数量

这张表的重点不是某一类具体分值，而是说明：

- 他们先建立 taxonomy，再做人工评测

##### 段落 3：如何构造安全测试集

作者强调他们不只追求内容类别多样，还追求提问形式多样：

- inducement
- role-playing
- multi-turn dialogues
- preset positions

最终得到：

- `2400` 个问题

##### 段落 4：如何标注

标注分三类：

- safe
- unsafe
- model refusal

并且做了 training 和 cross-verification。

##### 段落 5：作者对人工安全评测的结论

他们把“安全回答”和“合理拒答”都算 secure response，并认为模型在多类安全测试上表现良好。

##### 段落 6：再引入 `Do-Not-Answer`

除了自建安全评测，他们还用：

- `Do-Not-Answer`

这是 `939` 个风险提示组成的数据集。

##### `Table 11`：Do-Not-Answer 得分

作者报告：

- `DeepSeek-67B-Chat = 97.8`

并指出它高于：

- `ChatGPT = 97.7`
- `GPT-4 = 96.5`

#### 小结

这节的重点有两个：

- 安全不是只跑一个 benchmark，而是先定义问题空间，再做人工审查
- 拒答能力被明确纳入了好模型标准

### 5.5 Discussion

#### 这一小节在原文里做什么

这是整篇论文里非常值钱的一节，因为它不再只是报结果，而是在总结训练过程中真正踩出来的经验。

### 5.5.1 Staged Fine-Tuning

#### 按段落顺序梳理

##### 段落 1：问题定义

作者先说：

- 小模型需要在 math/code 数据上 fine-tune 更久
- 但这样会损害对话能力，比如提升 repetition

##### `Table 12`：两阶段 fine-tuning

表中对比：

- `7B Chat Stage1`
- `7B Chat Stage2`

关键结果：

- `HumanEval` 和 `GSM8K` 基本不掉
- repetition 从 `0.020` 降到 `0.014`
- `IFEval` 从 `38.0` 提到 `41.2`

##### 段落 2：作者结论

作者据此认为：

- 第二阶段不会明显伤 code/math
- 还能降低 repetition、提高 instruction following

### 5.5.2 Multi-Choice Question

#### 按段落顺序梳理

##### 段落 1：为什么研究 MC 数据

作者指出很多 benchmark 本身是多选题，所以他们测试：

- 在对齐阶段额外加入 `20M` 中文 MC 数据会怎样

##### `Table 13`：MC 数据的影响

加入 MC 数据后：

- `MMLU`、`C-Eval`、`CMMLU` 大涨
- 但 `TriviaQA`、`ChineseQA` 这种生成式评测几乎没提升，甚至略降

##### 段落 2：作者给出的结论

作者的态度很明确：

- MC 数据会提升“做多选题”的能力
- 但不等于模型在真实对话里变得更聪明

因此他们选择：

- 不在 pre-training 和 fine-tuning 里加入 MC 数据

这是很典型的“拒绝为 benchmark 过拟合”的选择。

### 5.5.3 Instruction Data in Pre-Training

#### 按段落顺序梳理

##### 段落 1：实验设定

作者提到业内常见做法：

- 在预训练后期混入 instruction data

他们试了：

- 在预训练最后 `10%` 阶段加入 `5M` instruction data
- 这些数据主要还是多选题

##### 段落 2：结果和判断

作者发现：

- 这样确实能让 base model benchmark 更好看
- 但最终效果和把同样数据放到 SFT 阶段几乎等价

所以他们的结论是：

- 这更像 benchmark 强化，不是潜力本身提升
- 如果 instruction data 非常大，可以考虑混入 pre-training
- 但在他们当前条件下，没有必要这样做

### 5.5.4 System Prompt

#### 按段落顺序梳理

##### 段落 1：先给 prompt

作者直接给出 system prompt，其目标是：

- helpful
- respectful
- honest
- safe

##### 段落 2：7B 和 67B 的反应不同

作者观察到：

- `7B` 加 system prompt 后 `MT-Bench` 略降：`7.15 -> 7.11`
- `67B` 加 system prompt 后明显提升：`8.35 -> 8.58`

##### `Table 14`：system prompt 的影响

这张表支撑的关键结论是：

- 大模型更能理解 system prompt 的意图
- 小模型更容易因为训练测试不一致而受负面影响

#### 小结

Discussion 这一节最值得带走的是四个实践判断：

- staged fine-tuning 可以同时顾及能力和对话质量
- MC 数据能刷分，但未必提升真实生成智能
- instruction data 混入 pre-training 不一定比放到 SFT 更有价值
- system prompt 的收益受模型规模强烈影响

## 6. Conclusion, Limitation, and Future Work

### 这一章在原文里做什么

最后一章做三件事：

- 回收整篇论文的核心贡献
- 承认模型当前限制
- 给出下一步方向

### 按段落顺序梳理

#### 段落 1：总结论文完成了什么

作者总结说：

- 发布了一系列从零训练的开源中英双语模型
- 训练数据规模是 `2T` tokens
- 系统解释了 hyperparameter selection、scaling laws 和 fine-tuning 尝试
- 校准了此前的 scaling laws
- 提出新的最优 model/data allocation 策略
- 给出在给定 compute budget 下预测 near-optimal batch size 和 learning rate 的方法
- 进一步指出 scaling law 与数据质量相关

这一段其实就是整篇论文的正式收口。

#### 段落 2：limitations

作者承认的限制包括：

- 预训练后知识不再持续更新
- 可能生成非事实内容
- 可能 hallucinate
- 中文数据第一版还不够完整
- 其他语言能力还比较脆弱

这一段很重要，因为它提醒我们不要把第一阶段神化成“已经无短板”。

#### 段落 3：future work

作者列出三条未来方向：

- 很快会发布 code intelligence 和 `MoE` 的技术报告
- 正在构建更大、更好的数据集，继续提升 reasoning、中文知识、math、code
- alignment 团队在研究如何把模型做得更 helpful、honest、safe，并且初步实验已经表明 RL 可能提升复杂推理能力

这个 future work 段现在回头看非常有意思，因为它几乎就是后续路线的预告。

### 小结

结论章最值得记的不是“模型很强”，而是：

- 第一阶段已经把后面两条主线预告出来了
- 一条通向 `MoE`
- 一条通向更强的 `RL reasoning`

## 附录里最值得看的两部分

### A.2 Different Model Scale Representations

这一部分最重要的是 `Figure 6`。

#### `Figure 6` 在说明什么

作者比较了不同模型规模表示方式的性能预测曲线，并得出：

- `6N1` 会高估大模型性能
- `6N2` 会低估大模型性能
- `M = non-embedding FLOPs/token` 的预测更准确

#### 为什么它重要

因为它不是“换个符号记法”，而是在修正：

> 我们到底应该怎样度量“模型规模”。

### A.3 Benchmark Metrics Curves

这一部分最值得看 `Figure 7`。

#### `Figure 7` 在说明什么

它展示 base model 在各 benchmark 上随训练推进的曲线。

#### 为什么它重要

这能帮助你判断：

- 当前训练终点到底更像“完全收敛”
- 还是“在工程上选了一个合理停点”

## 读完整篇后，最该带走的 12 个点

1. `DeepSeek LLM` 不是单纯的模型发布文，而是“scaling law + 真实训练决策落地”论文。
2. 摘要只负责交代问题、模型、流程和结果，不负责展开后文的 scaling 细节。
3. Introduction 的关键任务是说明为什么要重做 scaling law，而不是直接发模型。
4. 第一阶段的数据工程已经很重，尤其强调大范围去重、质量过滤和分布重混。
5. 第一阶段架构总体仍是 dense LLaMA 风格，重点不是花哨结构，而是稳定底座。
6. multi-step LR scheduler 的选择体现的是“可复用训练策略”，不只是单次最优。
7. Section 3 的核心贡献之一，是把 `batch size / learning rate` 也纳入 scaling law。
8. 另一个核心贡献，是用 `non-embedding FLOPs/token` 重新定义模型规模。
9. 数据质量不是旁支因素，它会直接改写最优 model/data allocation。
10. 第一阶段后训练仍是 `SFT + DPO`，但作者已经很重视 repetition 这类真实副作用。
11. Discussion 部分比很多结果表更值钱，因为它沉淀了 staged SFT、MC 数据、instruction mixing、system prompt 的实践判断。
12. 结论章里对 `MoE` 和 `RL` 的预告，正是后续路线真正展开的方向。

## 如何回到路线笔记

如果现在要把这篇论文重新放回整个 DeepSeek 路线里，可以只记三件事：

- 第一阶段真正解决的是 `scaling law`、训练配比和基础盘，而不是发明新架构。
- 后续 `V2` 之所以会走向 `DeepSeekMoE + MLA`，是因为第一阶段已经把 dense 路线该校准的地方先校准完了。
- 结论章里对 `MoE` 和更强 `RL` 的预告，基本就是后面 `V2 / R1 / V4` 的展开方向。

## 相关笔记

- [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md)
